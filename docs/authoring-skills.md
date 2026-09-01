# Authoring Skills That Change Agent Behavior

A skill that doesn't measurably change what an agent *does* is documentation, not a skill. This guide is the methodology this repo uses — TDD applied to process writing. It is distilled from practice and aligns with the [agentskills.io spec](https://agentskills.io).

## The Iron Law

> **No skill without a failing test first.**

Before writing a single line of the skill, watch an agent *fail* at the task without it. That baseline is your spec: it tells you exactly what the skill must teach, and later proves whether the skill taught it.

## The Cycle: RED → GREEN → REFACTOR

### RED — Baseline

1. Pick a realistic task that exercises the skill's core judgment (not a trivia question — a pressure scenario).
2. Run it on a **fresh agent with no access to the skill**.
3. Document verbatim: what they did, what they skipped, which rationalizations they used.

**Triage the failure type** — it determines the fix form:

| Baseline failure | Right fix form |
|---|---|
| Knows the rule, violates it under pressure | Prohibition + rationalization table + red flags list |
| Output exists but has the wrong shape | Positive recipe: state what the output IS, in order |
| Omits a required element | Structural slot in a template, not prose reminders |
| Behavior should depend on context | Conditional keyed to an observable predicate |

### GREEN — Write the minimal skill

Write only what addresses the observed baseline failures. Resist adding hypothetical cases. Two structural rules that matter more than they look:

**The description trap.** The frontmatter `description` is what agents match against to *decide whether to load* the skill. If it summarizes the workflow, agents follow the two-line summary and skip the body. Description = **triggering conditions only** ("Use when…"), never process.

**Word budgets.** Skills load into context. Frequently-loaded skills: <200 words. Reference skills: <500 words. Everything heavy goes in a linked supporting file the agent loads *when executing, not when deciding*.

### REFACTOR — Close loopholes

Re-run the scenario with the skill loaded. Any new rationalization the agent invents gets an explicit counter in the table. Re-test until the behavior converges. (If every rep produces the same shape, the wording is binding.)

## Testing Methods by Skill Type

| Skill type | Test with | Success looks like |
|---|---|---|
| Discipline (rules under pressure) | Pressure scenarios: time cost, sunk cost, authority, exhaustion | Agent follows the rule while naming the pressure |
| Technique (how-to) | Application to a *new* scenario + edge cases | Correct transfer, no gaps in instructions |
| Pattern (mental model) | Recognition + counter-examples | Knows when it does NOT apply |
| Reference (docs/APIs) | Retrieval + gap testing | Finds and correctly applies the right section |

**Micro-testing wording:** before full scenarios, verify wording with 5+ single-shot samples against a no-guidance control. Read every flagged match yourself — automated counting overstates both failures and successes. If the control doesn't exhibit the failure, there is nothing to fix: stop.

## Anatomy of a SKILL.md

```markdown
---
name: skill-name-with-hyphens        # letters, numbers, hyphens only
description: Use when <triggering conditions and symptoms>. Third person.
---                                  # NEVER summarize the workflow here (max 1024 chars total)

# Title

## Overview            # core principle in 1-2 sentences
## When to Use         # symptoms + explicit "NOT for" list
## The Workflow        # numbered, decision points as small flowcharts ONLY where non-obvious
## Hard Rules          # non-negotiables, stated as prohibitions with reasons
## Rationalizations — STOP   # | excuse | reality | — captured from RED phase, verbatim
## Red Flags           # self-check list: "if you're doing X, stop and start over"
## Quick Reference     # pointer to supporting files (load-when-executing)
```

### What separates professional-grade skills

1. **Every rule has a scar.** Rules exist because skipping them cost a real incident. "Pushed ≠ built ≠ served ≠ loaded" is weak advice; "we shipped a fix and the user's browser still ran the old chunk for an hour" is a law.
2. **Commands are executable, not illustrative.** Include the quoting pitfalls, the CRLF gotchas, the reserved-variable names — the things that cost the session time, not the happy path from the docs.
3. **Rationalizations are quoted, not invented.** Capture what agents (and humans) actually say when about to violate the rule. Invented objections bulletproof nothing.
4. **One excellent example beats five mediocre ones.** Complete, runnable, commented with *why*, from a real scenario.
5. **Supporting files separate judgment from execution.** SKILL.md = what to think and when. templates.md/scripts = what to run once you know.

## Anti-Patterns

- **Narrative skills** — "that time we found the bug" with no reusable procedure.
- **Workflow summaries in the description** — trains agents to skip the body.
- **Multi-language examples** — dilution; port one great example when needed.
- **Prohibitions for shape problems** — "don't restate the spec" produces more restating; state the target shape instead.
- **Unbounded scope** — if "When to Use" can't name what the skill is *not* for, it's too broad.
- **Shipping untested** — reading ≠ using. A skill that was never baselined is a guess with formatting.

## Submission Checklist

Before opening a PR to this repo:

- [ ] Baseline failure documented (what agents do WITHOUT the skill, verbatim)
- [ ] With-skill run shows the behavior change
- [ ] `description` starts with "Use when", contains no workflow summary, < 1024 chars total frontmatter
- [ ] Rationalization table entries are quoted from testing, not imagined
- [ ] Word budget respected; heavy material in supporting files
- [ ] Commands tested on at least one real environment; secrets never appear in examples
- [ ] "When NOT to use" section present
- [ ] No project-specific details that belong in the contributing project's own docs
