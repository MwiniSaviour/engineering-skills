# Changelog

All notable changes to this project are documented in this file.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows [SemVer](https://semver.org/), where MAJOR = skill format/spec breaking changes, MINOR = new or materially changed skills, PATCH = fixes and wording.

## [Unreleased]

### Added

- `ship-a-feature` skill: gated feature-delivery lifecycle - research, spec, plan, implement, verify, review, browser smoke, **approval gate**, promote, post-deploy verification. Per-deploy fresh-approval rule (silence and stale authorizations are not launch keys), final-tree evidence discipline, subagent delegation rules (a summary is a claim, not evidence; a bare LGTM = didn't look), served-code verification (pushed != built != served != loaded), rationalization and red-flag handling.
- `templates.md` operating reference for that skill: per-phase subagent prompts (research findings with file paths, severity-tagged review), final-tree verification commands, evidence-pack approval template, promote + rollback checklists.

## [1.0.0] - 2026-09-01

### Added

- `debugging-production-systems` skill: evidence-first debugging workflow for deployed/self-hosted systems — reproduction fidelity ladder (code → server API variant matrix → headless real-browser payload capture), deployed-vs-served-vs-browser verification, one-hypothesis-per-deploy discipline, boundary-first fixes, hardening pass.
- `templates.md` operational reference for that skill: SSH/credential discovery, prod DB via stdin piping with a quoting-pitfall table, deployed/bundle verification, nginx access-log forensics, third-party API variant matrices, Playwright payload-capture harness, GitHub Actions run watching, artifact hygiene.
- `docs/installation.md` — install guide for opencode, Claude Code, and generic Agent Skills runtimes, with verification smoke test.
- `docs/authoring-skills.md` — the RED/GREEN/REFACTOR methodology for writing skills that measurably change agent behavior.
- `docs/skill-template.md` — starter skeleton with the required sections.
- `CONTRIBUTING.md` — quality bar and submission process.
