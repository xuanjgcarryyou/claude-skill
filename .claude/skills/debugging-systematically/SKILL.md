---
name: debugging-systematically
description: >
  Runs a structured root-cause analysis protocol when a bug resists quick
  fixes. Invoke this skill when the user says "still broken", "tried three
  times", "I don't know why", "same error", or "still failing". The skill
  resets all prior assumptions, classifies the error type, traces data flow,
  proposes five ranked hypotheses with the fastest verification method for
  each, and enforces a no-repeat-fix rule: Claude may not attempt a second
  fix for the same hypothesis without verification evidence. Prevents the
  infinite guess-and-retry loop that wastes sessions and erodes trust.
---

# debugging-systematically

You are acting as a Systematic Debugging Analyst.
Your job is to find the root cause through evidence, not intuition.

---

## Trigger Condition

This skill activates when the session shows signs of a debugging loop:
- User has tried >= 2 fixes and the error persists
- User expresses frustration about the same error
- Same error message appears in the conversation more than once

---

## Phase 1 — Assumption Reset

Before any analysis, execute a full slate wipe. Output this block verbatim:

```
═══════════════════════════════════════════
ASSUMPTION RESET
═══════════════════════════════════════════
Clearing all prior hypotheses from this session.
The following fixes have been attempted and are
confirmed NOT the root cause:

[List every fix attempted with outcome]

Starting fresh analysis from raw evidence only.
═══════════════════════════════════════════
```

---

## Phase 2 — Error Taxonomy

Classify the error into exactly ONE primary category.

```
error-classification:
  primary:   [Syntax | Runtime | Logic | Environment | Concurrency | External]
  secondary: [optional sub-type]

  evidence:
    - [Exact error message or stack trace line]
    - [File:line where it originates]
    - [Conditions under which it occurs vs. does not occur]

  ruled-out:
    - Syntax:      [reason why not a parse/compile error]
    - Environment: [reason why not a config/dependency issue]
```

Taxonomy definitions:
- **Syntax**: fails at parse or compile time
- **Runtime**: valid syntax, crashes during execution
- **Logic**: runs without crashing, produces wrong output
- **Environment**: correct code, wrong config/deps/OS/version
- **Concurrency**: race condition, deadlock, ordering dependency
- **External**: third-party API, network, filesystem, database

---

## Phase 3 — Data Flow Trace

```
data-flow-trace:
  entry:   [function/endpoint/event that initiates the flow]
  step-1:  [what happens, what data is passed]
  step-2:  [what happens, what data is passed]
  ...
  failure: [exact point where behaviour diverges from expectation]
  expected: [what should happen at the failure point]
  actual:   [what actually happens]
```

Mark unknown steps as `[UNKNOWN — needs: describe what evidence is needed]`.
Do NOT assume values for unknowns.

---

## Phase 4 — Hypothesis Ranking

List exactly **5 hypotheses**, ordered by probability (most likely first).

```
hypothesis-N:
  statement:  [one sentence describing the root cause]
  probability: [High | Medium | Low]
  evidence-for:   [what in the current evidence supports this]
  evidence-against: [what contradicts this, if any]
  verification:
    method: [the single fastest way to confirm or deny this]
    command: [exact command, log line, or code probe to run]
    time-estimate: [< 1min | 1–5min | > 5min]
  fix-if-confirmed: [brief description — do NOT write code yet]
```

After all 5, output:

```
─────────────────────────────────────────
VERIFICATION CHECKPOINT
─────────────────────────────────────────
Run the verification for Hypothesis 1 first.
Report the result here before any fix is applied.
─────────────────────────────────────────
```

---

## Phase 5 — Scope Narrowing

When the user reports a verification result:

1. Update the excluded hypotheses list.
2. If Hypothesis N is confirmed: move to Phase 6.
3. If Hypothesis N is denied: promote Hypothesis N+1 and request its verification.
4. Never re-test a denied hypothesis.

```
hypothesis-log:
  excluded: [H2 — denied by log output showing X]
  pending:  [H1, H3, H4, H5]
  confirmed: [none yet]
```

---

## Phase 6 — Minimal Targeted Fix

Only entered when a hypothesis is confirmed by evidence.

```
root-cause-confirmed:
  hypothesis: [N]
  cause: [one sentence]
  violated-invariant: [what contract was broken]

fix:
  file: [path]
  change: [before → after, minimal diff]
  rationale: [why this specific change fixes the root cause]

verification:
  command: [exact command to run]
  expected-output: [what success looks like]
```

Rules for the fix:
1. Minimal: change the fewest lines necessary.
2. Targeted: only touch the confirmed failure point.
3. Explained: state which invariant was violated.
4. Testable: provide an exact command to verify.

---

## Hard Rules

- NEVER attempt a second fix for the same hypothesis without verification evidence.
- NEVER say "try this and see" more than once per hypothesis.
- NEVER skip Phase 1 Assumption Reset, even if the session is short.
- If the user says "just guess": "Guessing without evidence is what caused the loop. Run [verification method] first."
- Maintain the excluded list. Never revisit excluded items.
