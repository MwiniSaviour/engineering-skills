---
name: ship-a-feature
description: Use when implementing a feature end-to-end for eventual production release — when the user asks to build, add, or ship a feature and expects it delivered green and deployed, when work spans research/spec/plan/implement/verify/review/smoke-test phases, or when a feature is code-complete and needs the path to production. Also use when coordinating subagents or skills across a feature lifecycle.
---

# Ship a feature

## Overview

Ship a feature from request to production through gated phases. **Approval is per-deploy, fresh, and explicit — a morning authorization is not a launch key.** The feature is done when it is verified in production, not when tests pass.

## When to use

- The user asks for a feature built and shipped ("add X and deploy it", "get this out today")
- A feature is code-complete and needs the path to production (review, smoke, approval, promote)
- You are orchestrating phases, subagents, or skills across a feature lifecycle

**When NOT to use:**
- Bug fixes in shipped code — use `debugging-production-systems`
- Pure research, questions, or read-only exploration
- Trivial single-file edits with no deploy expectation
- Ops-only changes with no feature code

## The workflow

```
research → spec → plan → implement → verify → review → smoke → APPROVAL → promote → verify-prod
```

Each phase ends at a **gate** — a written artifact or command output, not a feeling. Gates cannot be skipped, only fast-forwarded when their output already exists from this session.

| Phase | Gate artifact | Fast-forward condition |
|---|---|---|
| 1. Research | Written findings: constraints, patterns, affected surfaces | User supplied the answers |
| 2. Spec | Intent + acceptance criteria, user-confirmed | User wrote the spec |
| 3. Plan | Every task verifiable | Trivial feature (≤2 tasks) |
| 4. Implement | Code + tests (TDD where logic exists) | — |
| 5. Verify | Suite + typecheck output **from the final tree** | — |
| 6. Review | Findings dispositioned: fixed / justified / ticketed | — |
| 7. Smoke | Browser/app evidence against a running build | — |
| 8. Approve | User's explicit go for **this** deploy, after the evidence | — |
| 9. Promote | Deploy output + post-deploy health checks | — |

**Phases 1–3 may compress for small features** (spec paragraph + 3-task plan in one message). **Phases 5–9 never compress.**

### Delegation rules

- Parallelize research/review/QA subagents when independent; implement via one focused subagent at a time.
- A subagent's summary is a **claim, not evidence** — re-run its key verification or spot-check its central claim.
- A bare "LGTM" with no specifics is indistinguishable from "didn't look" — demand files/flows examined and findings-by-severity, or re-dispatch.
- Load relevant skills per phase (`security-review` for auth, `database-migration` for schema) — descriptions are the router, not a formality.

## Hard rules

- **Never promote to production without a fresh, explicit approval for this deploy.** "Ship it today" at 9am is not approval at 9pm. Silence is never approval. The scar: a morning authorization treated as a launch key that survives the user going offline.
- **Evidence must come from the final tree.** A test run predating your last edit validates a tree that no longer exists. Re-run after the last change.
- **Every review finding gets a disposition** — fixed, justified-in-writing, or deferred with a ticket. "I'll fix it later" without a ticket means it ships.
- **Smoke-test against a running build, not a green CI badge.** CI proves compile, not behavior. The scar: a promote reused a staging-built app image while the seed image was never rebuilt — all gates green, production broke.
- **No `git add -A` on a shared tree.** Stage explicit paths. The scar: a user's uncommitted work riding along in an agent's commit.
- **Verify the deploy served the new code** (image SHA / build id / version endpoint) before declaring victory. Pushed ≠ built ≠ served ≠ loaded.

## Rationalizations — STOP

| Excuse | Reality |
|---|---|
| "The user said ship it today — that instruction doesn't expire when they stop replying" | Authorization is per-deploy. A deadline is urgency, not permission. Re-confirm at the gate. |
| "All the gates passed — CI, tests, staging — waiting would fail the deadline" | Gates passing is the *entry condition* for approval, not a substitute. |
| "It's a small change, the earlier test run still applies" | The run validated a tree that no longer exists. Re-run; it's minutes. |
| "The findings are in paths that should rarely trigger" | "Should rarely" is a frequency guess about a correctness bug. Retry paths run under degraded conditions — where race windows widen. |
| "The reviewer said LGTM" | A bare LGTM is a claim. Get specifics or re-dispatch. |
| "Smoke-testing is redundant, CI already built it" | CI proves compile, not behavior. Real outages have shipped through green CI. |
| "I'll fix the review finding in a follow-up" | Without a ticket, "follow-up" is where findings go to die. Fix it or ticket it. |

## Red flags — STOP

- About to run a production deploy and cannot quote the user's approval for **this** deploy
- Your "evidence" cites a run predating your most recent edit
- Your report says "should", "probably", or "assumed" where command output belongs
- A review returned findings your diff since doesn't address
- About to `git add -A` / `git add .` on a tree that was dirty before you started
- The deployed SHA does not match the SHA you verified

## Quick reference

Operating detail — subagent prompts, gate commands, deploy verification, rollback:
[templates.md](templates.md). Load when executing, not when deciding.
