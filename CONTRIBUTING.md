# Contributing

Skills in this repo are **tested operating procedures**, not documentation. The bar for merging is behavioral evidence, and the process below is how we get it.

## The non-negotiable

> **No skill ships without baseline evidence.** Before writing the skill, watch an agent fail at the task without it. After writing it, watch an agent succeed with it. The full methodology — RED/GREEN/REFACTOR for documentation, failure-type triage, micro-testing — is in [docs/authoring-skills.md](docs/authoring-skills.md).

## Adding a new skill

1. **Open an issue first** describing the failure mode the skill addresses: what agents get wrong, with a concrete scenario. If you can't write the failing scenario, the skill isn't ready to exist.
2. **Baseline (RED):** run the scenario on a fresh agent without the skill. Save the transcript (redact any secrets). Summarize verbatim rationalizations.
3. **Write the skill (GREEN):** follow the anatomy in the authoring guide. `SKILL.md` carries judgment (workflow, rules, rationalizations); heavy command reference goes in a supporting file (e.g. `templates.md`) that is loaded *when executing*.
4. **Verify (REFACTOR):** re-run the scenario with the skill. New rationalizations → new counters in the table → re-test. Note the results in the PR.
5. **Submit:**
   - `skills/<skill-name>/SKILL.md` (+ supporting files)
   - A one-paragraph **Provenance** note for the README table: the real-world incident or repeated failure that justifies the skill's existence
   - An entry in `CHANGELOG.md` under `## [Unreleased]`

## Modifying an existing skill

The Iron Law applies to edits too: run the baseline *with the current skill* to document the gap, then apply RED/GREEN/REFACTOR to your change. "It's just a wording tweak" is how binding skills become loose ones.

## Style requirements

- Frontmatter `description` = triggering conditions only ("Use when…"), never a workflow summary
- Word budget: judgment content in SKILL.md stays lean (< ~500 words); supporting files carry the bulk
- Commands must be tested on a real environment; placeholder hosts/paths (`SERVER_IP`, `/opt/APP`), never real credentials or client names
- Every prohibition gets a reason — a rule without a scar reads like preference
- Tables over prose for scanning; flowcharts only for genuinely non-obvious decision loops
- American English, sentence-case headings after the title

## What we will not merge

- Skills without baseline evidence ("it's obviously useful" is not evidence)
- Workflow summaries in descriptions
- Narrative war stories without a reusable procedure
- Anything containing real secrets, customer data, or unredacted infrastructure details
- Project-specific conventions that belong in that project's own agent config

## Review process

A maintainer reproduces your with-skill verification run (same scenario, fresh agent) before merging. PRs should include the baseline + verification transcripts or summaries as PR description sections: **Baseline (no skill)** / **Verification (with skill)** / **Rationalizations captured**.
