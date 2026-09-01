# Production Debugging — Template Commands

Load when EXECUTING the workflow, not when deciding. All paths/hosts are placeholders: `SERVER_IP`, `/opt/APP`, `DOMAIN`, `OWNER/REPO`, `SERVICE`, `ENVFILE` (usually `.env.production`).

## 1. Credential & Access Discovery

Find how to reach the server and GitHub before anything else.

```powershell
# SSH: what keys exist? Which hosts are known? (known_hosts = IP/hostname inventory)
Get-ChildItem "$env:USERPROFILE\.ssh"
Get-Content "$env:USERPROFILE\.ssh\known_hosts" | ForEach-Object { ($_ -split ' ')[0] } | Select-Object -Unique

# Test access (bounded, auto-accept host key on first connect)
ssh -i "$env:USERPROFILE\.ssh\KEY_NAME" -o StrictHostKeyChecking=accept-new -o ConnectTimeout=15 root@SERVER_IP "hostname && uptime"
```

```powershell
# GitHub: token from the local credential store — never hardcode PATs in scripts
$cred  = "protocol=https`nhost=github.com`n" | git credential fill
$token = ($cred | Select-String '^password=').Line.Substring(9)
# If deploy coords aren't local, read the deploy workflow for host/user/key names:
# .github/workflows/deploy.yml — secrets.VPS_HOST / VPS_USER / VPS_SSH_KEY map to what's in ~/.ssh
```

## 2. SSH + Docker Compose Patterns

```bash
# Service inventory + health (run ON the server)
ssh -i KEY root@SERVER_IP 'cd /opt/APP && ls docker-compose.yml && docker compose --env-file ENVFILE ps'
# Service logs (time-boxed)
ssh ... 'cd /opt/APP && docker compose --env-file ENVFILE logs app --since 30m | grep -iE "error|timeout" | tail -20'
```

### Quoting pitfalls (PowerShell → ssh → bash → docker) — read before writing any command

| Symptom | Cause | Fix |
|---|---|---|
| `psql: extra command-line argument ignored` / `role "\" does not exist` | Remote command got split/expanded across shell layers | Never inline complex SQL in the command string — pipe it via stdin (§3) |
| `bash: line 5: $'echo\r': command not found` | PowerShell's pipeline re-adds CRLF to stdin | Strip CR before writing: `[IO.File]::WriteAllText($path, ($script -replace "\`r",""))`; if still corrupt, `scp` the file then `ssh ... 'bash /tmp/x.sh'` |
| `head`/`Select-String: not recognized` in output | Cmdlets ran LOCALLY on ssh output | All pipes/filters must live INSIDE the remote command string |
| `column "schoolid" does not exist` | Postgres folds unquoted identifiers to lowercase | Double-quote camelCase: `"schoolId"`, `"PaymentConfig"` |
| `$VARIABLES` arrived empty/expanded | PowerShell expanded them before ssh saw them | Single-quote the remote command; `$$` escapes inside docker `sh -c` |

## 3. Production DB Queries (stdin pipe pattern)

```powershell
# Single query: write locally, pipe via stdin — quoting survives every layer
Write-Output 'SELECT "schoolId", "subaccountCode" FROM "PaymentConfig" LIMIT 5;' | Out-File -Encoding utf8 -NoNewline "$env:TEMP\opencode\q.sql"
Get-Content "$env:TEMP\opencode\q.sql" -Raw | ssh -i KEY root@SERVER_IP 'cd /opt/APP && docker compose --env-file ENVFILE exec -T db psql -U postgres -d DBNAME'

# Multi-query script: here-string → strip CR → stdin
$sql = @'
SELECT "reference", status FROM "Transaction" WHERE type = 'FEE_PAYMENT' ORDER BY "createdAt" DESC LIMIT 8;
SELECT slug, "planTier", "planStatus" FROM "School" WHERE slug = 'SLUG';
'@
[IO.File]::WriteAllText("$env:TEMP\opencode\q2.sql", ($sql -replace "`r",""))
Get-Content "$env:TEMP\opencode\q2.sql" -Raw | ssh ... 'cd /opt/APP && docker compose --env-file ENVFILE exec -T db sh -c ''psql -U "$POSTGRES_USER" -d "$POSTGRES_DB"'''
# (the sh -c form resolves DB creds from the compose env — use when names are unknown)
```

## 4. "What is ACTUALLY deployed?" — build/bundle verification

```bash
# Inside the container: build ID + does the compiled bundle contain the new code?
ssh ... 'cd /opt/APP && docker compose --env-file ENVFILE exec -T app sh -c "cat .next/BUILD_ID; echo; grep -rl \"NEW_CODE_MARKER_STRING\" .next/static/chunks/ 2>/dev/null | head -3"'
```

```powershell
# Over HTTPS: is the SERVED bundle the same build? (fingerprinted chunk path from the grep above)
$c = Invoke-WebRequest -Uri 'https://DOMAIN/_next/static/chunks/.../page-HASH.js' -UseBasicParsing
('status: ' + $c.StatusCode + ' len=' + $c.Content.Length)
('contains new code: ' + ($c.Content -match 'NEW_CODE_MARKER_STRING'))

# Live response headers (build id, CSP, etc.)
$h = (Invoke-WebRequest -Uri 'https://DOMAIN/' -UseBasicParsing).Headers
$h['X-Build-Id']
# Note: avoid variable name $home in PowerShell — it is reserved.
```

## 5. "What is the USER actually running?" — proxy log forensics

```bash
# Which fingerprinted chunk did the user's browser fetch, from which IP/UA/when?
# Old hash requested → stale tab/PWA. New hash → user is on current code.
ssh ... "grep 'parent/fees/page-' /var/log/nginx/access.log | tail -5"

# When did the user last hit the failing endpoint, from which device, what status + body size?
ssh ... "grep 'POST /api/endpoint' /var/log/nginx/access.log | tail -5"
# (body size hints at response shape: a response growing by ~50B ≈ the fields you added are present)
```

## 6. Third-Party API Variant Matrix (curl from the VPS)

Secrets stay on the server: grep them, never echo.

```bash
# Write the script locally, scp, execute — byte-safe (avoids §2 CRLF issues)
SECRET=$(grep '^PROVIDER_SECRET_KEY=' ENVFILE | cut -d= -f2-)
PUB=$(grep '^PROVIDER_PUBLIC_KEY=' ENVFILE | cut -d= -f2-)

probe() { LABEL="$1"; BODY="$2"
  echo "=== $LABEL ==="
  curl -s -X POST https://api.provider.com/init -H "Authorization: Bearer $SECRET" -H "Content-Type: application/json" -d "$BODY" > /tmp/init.json
  grep -o '"message":"[^"]*"\|"reference":"[^"]*"' /tmp/init.json
  ACC=$(grep -o '"access_code":"[^"]*"' /tmp/init.json | cut -d'"' -f4)
  curl -s -X POST https://api.provider.com/confirm -H "Authorization: Bearer $PUB" -H "Content-Type: application/json" \
    -d "{\"key\":\"$PUB\",\"code\":\"$ACC\",...}" | grep -o '"status":[a-z]*\|"message":"[^"]*"'
}
probe "A: exact app payload"     '{"email":"x@y.z","amount":100,"currency":"GHS",...}'
probe "B: minus one param"       '{"email":"x@y.z","amount":100,...}'
# Change ONE variable per row. Label refs DIAG-A-1 etc. so test artifacts are findable/archivable.
```

## 7. Headless Real-Browser Capture Harness (the decisive technique)

When server-side calls pass but the real client fails: run the vendor's actual widget JS in a real browser and capture what it sends.

```powershell
# One-time setup
npx playwright install chromium
```

```js
// pw-capture.js — run:  $env:NODE_PATH="<repo>\node_modules"; node pw-capture.js <args>
const { chromium } = require("playwright");
(async () => {
  const browser = await chromium.launch({ headless: true });
  const page = await browser.newPage();
  const captured = [];
  page.on("request",  req  => { if (req.url().includes("TARGET_ENDPOINT")) captured.push({ phase: "request", url: req.url(), postData: req.postData() }); });
  page.on("response", async res => { if (res.url().includes("TARGET_ENDPOINT")) { let body = ""; try { body = await res.text(); } catch {} captured.push({ phase: "response", status: res.status(), body: body.slice(0, 500) }); } });
  page.on("console",  msg  => { if (["error","warning"].includes(msg.type())) captured.push({ phase: "console", type: msg.type(), text: msg.text().slice(0, 300) }); });

  await page.goto("https://example.com", { waitUntil: "domcontentloaded" }); // neutral origin isolates the vendor from your bundle
  await page.addScriptTag({ url: "https://cdn.vendor.com/widget.js" });
  const result = await page.evaluate(() => new Promise(resolve => {
    try {
      // Drive the vendor's real API exactly as production does:
      const h = window.VendorWidget.setup({ /* EXACT production config fields */ });
      h.open();
      setTimeout(() => resolve({ outcome: "timeout-still-open" }), 15000);
    } catch (err) { resolve({ outcome: "setup-threw", error: String(err) }); }
  }));
  console.log("=== OUTCOME ===\n" + JSON.stringify(result, null, 2));
  console.log("=== CAPTURED ===\n" + JSON.stringify(captured, null, 2));  // postData = the ground truth
  await browser.close();
})().catch(e => { console.error("FATAL", e); process.exit(1); });
```

Use it twice: once to **find** the bug (captured payload vs. assumed payload), once to **prove the fix** (same harness, new config → expect the vendor's 200). NODE_PATH is required when the script lives outside the repo.

## 8. CI / Deploy Run Watching (GitHub API)

```powershell
$cred  = "protocol=https`nhost=github.com`n" | git credential fill
$token = ($cred | Select-String '^password=').Line.Substring(9)
$H = @{Authorization = "token $token"}

# Recent runs (a fresh push fires both CI and Deploy)
Invoke-RestMethod -Uri "https://api.github.com/repos/OWNER/REPO/actions/runs?per_page=3" -Headers $H |
  Select-Object -ExpandProperty workflow_runs |
  ForEach-Object { "{0} | {1} | {2} | {3} | {4}" -f $_.id, $_.name, $_.head_sha.Substring(0,7), $_.status, $_.conclusion }

# Which step failed? (jobs → steps with conclusion)
(Invoke-RestMethod -Uri "https://api.github.com/repos/OWNER/REPO/actions/runs/RUN_ID/jobs" -Headers $H).jobs |
  ForEach-Object { $_.steps | Where-Object conclusion -eq 'failure' | ForEach-Object { "{0} :: {1}" -f $_.name, $_.name } }

# Full logs for the failing run
Invoke-RestMethod -Uri "https://api.github.com/repos/OWNER/REPO/actions/runs/RUN_ID/logs" -Headers $H -OutFile "$env:TEMP\opencode\ci.zip"
Expand-Archive "$env:TEMP\opencode\ci.zip" "$env:TEMP\opencode\ci" -Force
# Then search locally: FAIL / ✕ / "Test Files" / the exact error string
```

## 9. Hygiene

- **Labeled test artifacts**: `DIAG-<variant>-<n>` references, `/tmp/<purpose>.json` on the server — everything findable and deletable.
- **Test transactions on provider dashboards**: archive/note them; on test mode they're free noise but pollute future debugging.
- **Temp files**: delete `%TEMP%\opencode\*` and server `/tmp` diagnostics at the end of every batch.
- **Never echo secrets** into logs, commands, or transcripts. Grep in place on the server.
