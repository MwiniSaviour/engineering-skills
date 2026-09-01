# Installation

Skills follow the open [Agent Skills format](https://agentskills.io): a directory containing a `SKILL.md` with YAML frontmatter (`name`, `description`) plus optional supporting files. Every runtime below reads the same directory — installation is copying it to the right place.

## 1. Clone the repo (all methods)

```bash
git clone https://github.com/MwiniSaviour/engineering-skills.git
cd engineering-skills
```

## 2. Copy the skill to your runtime's skills directory

### opencode

| OS | Skills directory |
|---|---|
| Windows | `%USERPROFILE%\.config\opencode\skills\` |
| macOS / Linux | `~/.config/opencode/skills/` |

```powershell
# Windows (PowerShell)
Copy-Item -Recurse skills\debugging-production-systems "$env:USERPROFILE\.config\opencode\skills\"
```

```bash
# macOS / Linux
cp -r skills/debugging-production-systems ~/.config/opencode/skills/
```

### Claude Code

| OS | Skills directory |
|---|---|
| Windows | `%USERPROFILE%\.claude\skills\` |
| macOS / Linux | `~/.claude/skills/` |

```bash
cp -r skills/debugging-production-systems ~/.claude/skills/
```

### Any other Agent Skills runtime

The spec is runtime-agnostic. Most runtures read from one or more of:

- `~/.agents/skills/` — user-global
- `<project>/.agents/skills/` — project-local (ships with the repo, applies to collaborators)

```bash
cp -r skills/debugging-production-systems ~/.agents/skills/
# or, to make it available to everyone on the project:
cp -r skills/debugging-production-systems /path/to/your/project/.agents/skills/
```

> If your runtime documents a different skills path, any of the above directories copied there works the same — the format is identical.

### Optional: skills CLI ecosystem

If you use a skills manager CLI, this repo is a standard Git source:

```bash
npx skills add https://github.com/MwiniSaviour/engineering-skills
```

(Availability in registry search depends on the CLI's index; direct Git add always works.)

## 3. Verify the install

1. Start a **fresh** session in your agent (skills are indexed at startup).
2. Ask it to list available skills, or simply check that it can see `debugging-production-systems`.
3. Smoke test — paste a scenario and confirm the skill's workflow appears:

   > Users report our hosted checkout iframe showing "Currency not supported by merchant" in production. Our curl tests against the payment API all pass. How do you proceed?

   A correct load looks like: the agent **refuses to conclude from curl alone**, proposes the reproduction-fidelity ladder (code → server API matrix → real browser payload capture), and reaches for the deployed-vs-served checks (`BUILD_ID`, chunk hashes, proxy logs) before touching code.

## 4. Updating

```bash
cd engineering-skills && git pull
# re-copy the skill directory over the installed one
cp -r skills/debugging-production-systems <your-runtime-skills-dir>/
```

## 5. Uninstalling

Delete the skill directory from the runtime's skills folder. Nothing else is installed — no daemons, no hooks, no dependencies.
