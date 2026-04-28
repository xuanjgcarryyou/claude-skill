# debugging-systematically

Systematic Root Cause Analyst — resets all assumptions and finds the root cause through evidence, not intuition.

## When to Use

Say any of the following to activate:
- "still broken"
- "tried three times"
- "I don't know why"
- "same error"
- "still failing"
- "I've tried everything"

Activates when the session shows: ≥ 2 failed fixes, user frustration, or the same error message appearing more than once.

## What It Does

Six phases, always in order:

| Phase | Action |
|-------|--------|
| 1 — Assumption Reset | Lists every fix attempted, clears all prior hypotheses |
| 2 — Error Taxonomy | Classifies error: Syntax / Runtime / Logic / Environment / Concurrency / External |
| 3 — Data Flow Trace | Traces from entry point to failure, marks unknowns explicitly |
| 4 — 5 Ranked Hypotheses | Lists 5 root cause hypotheses with probability and fastest verification method |
| 5 — Scope Narrowing | Updates excluded list as verifications come in; never re-tests denied hypotheses |
| 6 — Minimal Targeted Fix | Applied only after a hypothesis is confirmed by evidence |

## Verification Checkpoint

After Phase 4, the skill outputs a **VERIFICATION CHECKPOINT** block and stops.
It will not apply a fix until you run the verification and report the result.

## Hard Rules

- Never attempt the same hypothesis twice without verification evidence
- Never say "try this and see" more than once per hypothesis
- Maintains an excluded list — denied hypotheses are never revisited
- "just guess" → declined, skill redirects to the verification method
