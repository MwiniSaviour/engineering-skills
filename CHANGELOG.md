# Changelog

All notable changes to this project are documented in this file.
The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versioning follows [SemVer](https://semver.org/), where MAJOR = skill format/spec breaking changes, MINOR = new or materially changed skills, PATCH = fixes and wording.

## [Unreleased]

### Added

- `ship-a-feature` skill: gated feature lifecycle from request to production — research → spec → plan → implement → verify → review → smoke → approval → promote → verify-prod, where each phase ends at a gate (a written artifact or command output, not a feeling). Core rule: approval is per-deploy, fresh, and explicit — a morning authorization is not a launch key; silence is never approval. Includes delegation rules (subagent summaries are claims not evidence; bare "LGTM" = didn't look) and served-code verification (pushed ≠ built ≠ served ≠ loaded).
- `templates.md` operational reference for that skill: research/review subagent prompts, gate commands, the approval-request message template, promote + served-SHA verification, rollback-first discipline.

## [1.0.0] - 2026-09-01

### Added

- `debugging-production-systems` skill: evidence-first debugging workflow for deployed/self-hosted systems — reproduction fidelity ladder (code → server API variant matrix → headless real-browser payload capture), deployed-vs-served-vs-browser verification, one-hypothesis-per-deploy discipline, boundary-first fixes, hardening pass.
- `templates.md` operational reference for that skill: SSH/credential discovery, prod DB via stdin piping with a quoting-pitfall table, deployed/bundle verification, nginx access-log forensics, third-party API variant matrices, Playwright payload-capture harness, GitHub Actions run watching, artifact hygiene.
- `docs/installation.md` — install guide for opencode, Claude Code, and generic Agent Skills runtimes, with verification smoke test.
- `docs/authoring-skills.md` — the RED/GREEN/REFACTOR methodology for writing skills that measurably change agent behavior.
- `docs/skill-template.md` — starter skeleton with the required sections.
- `CONTRIBUTING.md` — quality bar and submission process.

