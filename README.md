# engineering-skills

> Production-grade skills (operating procedures) for AI coding agents — starting with evidence-first debugging of deployed systems.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Skills Spec](https://img.shields.io/badge/format-agentskills.io-purple)](https://agentskills.io)
[![Tested](https://img.shields.io/badge/method-RED%20%2F%20GREEN%20%2F%20REFACTOR-green)](docs/authoring-skills.md)

## What is this?

A curated collection of [Agent Skills](https://agentskills.io) — structured operating procedures that AI coding agents (opencode, Claude Code, and any spec-compatible runtime) load and follow when a task matches the skill's trigger conditions.

Think of a skill as the difference between an engineer who has *read about* production debugging and one who has *run the playbook*: what to check first, which commands to run, what counts as evidence, and which rationalizations to refuse under pressure.

## Included skills

### [`debugging-production-systems`](skills/debugging-production-systems/)

Evidence-first debugging for bugs in deployed/self-hosted systems — third-party widget errors, "works locally but fails in prod", stale deploys and browser caches, multi-deploy debugging spirals.

**Core principle:** *evidence before fixes, and match reproduction fidelity to the failure.*

```
The Reproduction Fidelity Ladder

  read code paths end-to-end (including the third party's own JS)
        │  still unexplained?
        ▼
  API variant matrix from the server (ONE param changed per request)
        │  API passes but users still fail?
        ▼
  headless real browser + the vendor's actual widget JS
  capture the request payload it really sends ◄── the smoking gun lives here
        │
        ▼
  prove the fix with the SAME harness that found the bug
```

What's inside:

| File | Contents |
|---|---|
| [`SKILL.md`](skills/debugging-production-systems/SKILL.md) | The workflow, hard rules, rationalization table, red flags |
| [`templates.md`](skills/debugging-production-systems/templates.md) | Operational commands: SSH/credential discovery, prod DB via stdin, deployed-vs-served verification, nginx forensics, third-party API variant matrices, a Playwright payload-capture harness, CI/deploy watching, hygiene rules |

**Provenance:** born from a real production incident — a payment funnel where `curl` against the provider's API passed every variant while every real browser failed, because the vendor's popup silently injected `currency: "NGN"` into requests. No amount of API-level testing could find that; only running the vendor's own JavaScript in a real browser could. The skill encodes that lesson and the full workflow around it.

## Install

Skills are plain directories — installation is a copy. One-liners:

```powershell
# opencode (Windows)
git clone https://github.com/MwiniSaviour/engineering-skills.git
Copy-Item -Recurse engineering-skills\skills\debugging-production-systems "$env:USERPROFILE\.config\opencode\skills\"
```

```bash
# Claude Code / generic Agent Skills runtimes
git clone https://github.com/MwiniSaviour/engineering-skills.git
cp -r engineering-skills/skills/debugging-production-systems ~/.claude/skills/
```

Full guide for every runtime (opencode, Claude Code, generic agentskills.io runtimes, project-local installs) plus verification steps: **[docs/installation.md](docs/installation.md)**.

## For skill authors

This repo doubles as a reference for engineers who want to write skills that actually change agent behavior instead of decorating it:

- **[docs/authoring-skills.md](docs/authoring-skills.md)** — the methodology: RED/GREEN/REFACTOR applied to documentation, frontmatter spec (and the "description summarizes workflow" trap), word budgets, rationalization tables, how to baseline-test on a fresh agent.
- **[skills/_template/SKILL.md](skills/_template/SKILL.md)** — starter skeleton.
- **[CONTRIBUTING.md](CONTRIBUTING.md)** — the quality bar: no skill ships without baseline evidence.

## Why "verify, then verify again"

Every rule in these skills exists because skipping it cost a real debugging session:

- *Pushed ≠ built ≠ served ≠ loaded* — four hops between `git push` and a user's browser, each independently stale-able.
- *Curl passes ≠ the user's browser passes* — vendor widget code transforms your config before it hits the wire.
- *The previous session's summary is a hypothesis* — grep the tree.

## License

[MIT](LICENSE) © 2026 MwiniSaviour
