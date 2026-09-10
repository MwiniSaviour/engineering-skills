# ship-a-feature — operating reference

Load this file when executing a phase, not when deciding whether to use the skill.
All hosts, paths, and project names are placeholders — substitute your project's real values.

## Phase 1 — research (gate: written findings)

Dispatch one explore/research subagent (or do it yourself for small features). The prompt must demand **written findings**, not a summary opinion:

```
Research task (no code changes). Map the affected surfaces for <feature>:
1. Existing patterns to follow (find the closest analog feature; name files)
2. Files/modules this touches (list paths)
3. Constraints (schema, API contracts, auth boundaries, term conventions)
4. Risks / unknowns worth flagging to the implementer
Return findings as a numbered list with file paths. Cite line numbers for claims.
```

Gate check: the findings exist in writing, with file paths. If the answer is "looks fine", it failed.

## Phase 2 — spec (gate: confirmed intent + acceptance criteria)

Write a short spec and **get user confirmation before implementing**:

```
Spec: <feature>
Intent: <one paragraph — what this does and why>
Acceptance criteria:
1. <observable behavior>
2. <observable behavior>
3. <edge case>
Out of scope: <explicit non-goals>
Confirm or correct before I plan the implementation.
```

## Phase 3 — plan (gate: verifiable task list)

Every task must be verifiable — "write the import parser" is not; "import parser passes the 6 fixture cases" is.

```
Plan: <feature>
1. <task> — verify: <command or observable>
2. <task> — verify: <command or observable>
3. ...
```

## Phase 4 — implement (gate: code + tests)

- TDD where logic exists: failing test → code → green. UI-only changes: component tests for behavior.
- One focused subagent at a time for implementation; never two writers on one tree.
- Commit at coherent checkpoints with explicit paths (`git add <paths>`, never `-A`).

## Phase 5 — verify (gate: final-tree evidence)

Run from the tree exactly as it will be committed — after the last edit:

```bash
# full suite + typecheck — record the output, cite it in the report
npm test            # or the project's suite command
npx tsc --noEmit    # or the project's typecheck
```

If anything changed after this run — even a message reword — re-run. The run is only evidence for the tree it ran on.

## Phase 6 — review (gate: dispositioned findings)

Dispatch a reviewer subagent with the spec + plan + diff. Demand severity-tagged findings:

```
Review task. You are reviewing a completed feature against its spec.
Spec: <paste>
Plan: <paste>
Diff/files: <paths or paste>
Return findings tagged [critical]/[warning]/[note] with file:line and a concrete fix for each.
Cover: spec compliance, error handling, concurrency, trust boundaries, resource leaks,
performance (N+1s), and test coverage gaps. "No issues" requires naming what you checked.
```

Disposition every finding: **fixed** (then re-verify), **justified** (written reason, user-visible), or **deferred** (ticket/issue created, named in the report). A finding with no disposition ships.

## Phase 7 — smoke (gate: running-build evidence)

Test the feature against a running build (staging or local dev) with a real browser or HTTP client:

- Exercise the acceptance criteria as a user would — click the flow, submit the form, read the response.
- Capture evidence: response codes, screenshots, DB row counts before/after.
- Verify the build you smoke is the build you'll ship (image tag / build id / version endpoint).
- Server-side assertions beat UI assertions where both exist (DB state after a UI action).

## Phase 8 — approval (gate: explicit go for this deploy)

Present the evidence and stop:

```
<Feature> is ready to ship:
- Verify: <suite> N/N pass, typecheck clean (run at <time> on <sha>)
- Review: <findings fixed/justified/deferred with tickets>
- Smoke: <what was exercised, against <build id>, evidence: <codes/screenshots/counts>>
- Deploy: <what promoting entails — image, migration, rollback path>
Approve the production deploy?
```

Then wait. Silence is not approval. A morning "ship it today" is not approval at the gate. If the user approves, quote their approval in the deploy report.

## Phase 9 — promote + verify-prod (gate: served-code proof)

```bash
# deploy via the project's mechanism (workflow dispatch, script, push-to-deploy)
# then verify the served code is the code you verified:
curl -s https://APP_HOST/api/health           # expect 200 + version/build id
docker ps --format "{{.Image}} | {{.Status}}"  # image tag matches promoted SHA
# exercise the feature's happy path once against production
```

Pushed ≠ built ≠ served ≠ loaded. Verify the served SHA, then run one production smoke of the feature's core path. Report: promoted SHA, health output, smoke result, rollback command.

## Rollback

Know the rollback path **before** promoting (it shapes the approval message). If the deploy serves wrong or the smoke fails: execute rollback first, investigate second — a broken production is not a debugging environment.
