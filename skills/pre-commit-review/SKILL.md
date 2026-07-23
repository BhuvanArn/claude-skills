---
name: pre-commit-review
description: >
  Pre-commit review of the current diff — checks that it is the RIGHT change, not just
  a change that passes. Use BEFORE any non-trivial commit, when the user says "review",
  "check before committing", "is this the right change", or asks whether the work is ready
  to commit/push. Also use it spontaneously at the end of a task, before proposing a commit —
  even if the user does not ask explicitly. Complements bug-hunting code review (e.g.
  /code-review): here we judge scope, contracts, simplicity, and alignment with intent.
---

# Pre-commit review — "is this the right change?"

CI proves the planned checks pass. **Not that we checked the right things.** This review
judges the change itself, before line-by-line reading.

## Step 0 — Establish intent

Before judging the diff, reconstruct what the change was *supposed* to do:

1. The original request (user message, issue/ticket, roadmap milestone, acceptance criteria).
2. The context: what is currently in progress, and what scope guardrails apply (design docs,
   ADRs, contributing guide, "what this project is NOT").

Without clear intent, the review cannot judge alignment — ask the user for it rather than
guessing.

## Step 1 — Collect the diff

```bash
git status --short
git diff HEAD          # or git diff <base>..HEAD if commits already exist
<run the project's test suite>   # test state BEFORE judging
```

Record the test result now — a green suite that was already green before the change proves
nothing about the change.

## Step 2 — The 9 questions

Answer each **with evidence pulled from the diff** (`file:line`), not an impression.
A question you cannot answer verifiably is a point to dig into, not one to tick off.

| # | Question | What it means |
|---|----------|---------------|
| 1 | **Need** | Are the acceptance criteria covered? Re-read the request, list each criterion, point to where the diff satisfies it. |
| 2 | **Scope** | Why was each file changed? Every file in the diff must be justified by the need. An unexplained file is a flag (scope creep, or a regenerated/artifact file committed by mistake). |
| 3 | **Contracts** | Did any existing API, schema, or behavior change? Public interfaces, data/serialization formats, config keys, DB migrations, wire formats. A contract change must be called out — and marked `BREAKING CHANGE:` if it breaks existing consumers or stored data. |
| 4 | **Consistency** | Does the diff follow the project's conventions (contributing guide, linter/formatter config, ADRs, established patterns)? |
| 5 | **Tests** | Would they have failed *before* the change? Do they verify behavior, not implementation? A test that already passed before the fix proves nothing. |
| 6 | **Security** | Inputs, permissions, secrets: no credentials/tokens/keys in the diff; no hardcoded machine-specific paths; caution around any code that reads untrusted input (files, network, user data). |
| 7 | **Operability** | Are errors, logs, and rollback understandable? Does a failure tell a readable story? Is a corrupted/partial state recoverable ("back out, don't destroy")? |
| 8 | **Simplicity** | Was there a simpler solution? Actively look: less abstraction, fewer files, reuse what exists. Prefer the minimum that solves the problem. |
| 9 | **Alignment** | Does the change respect the OVERALL intent, not just the local task criteria? Re-read the design pillars and scope guardrails: a change can tick every box and still pull the project off its trajectory. |

## Step 3 — Verdict

Return the result in this exact format:

```
## Review: <one-line summary of the change>

**Verdict: READY TO COMMIT | FIXES NEEDED | DISCUSSION REQUIRED**

| Question | ✅/⚠️/❌ | Evidence / problem |
|----------|--------|--------------------|
| 1. Need | ✅ | ... |
| ... | | |

### Blockers
(what prevents the commit — empty if none)

### Suggestions
(non-blocking improvements — user's discretion)
```

Verdict rules:

- ❌ on questions 1–5 or 9 → **FIXES NEEDED**. Fix, then re-run the review.
- Ambiguity about intent (question 9 undecidable) → **DISCUSSION REQUIRED**: present the
  possible interpretations to the user instead of choosing one silently.
- Suggestions never block: list them honestly, let the user decide.
- Don't pad the review: if the diff is good, say so in one table and stop.
