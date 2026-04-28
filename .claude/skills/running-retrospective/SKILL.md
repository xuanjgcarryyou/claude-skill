---
name: running-retrospective
description: >
  Conducts a structured post-feature retrospective that transforms this
  session's AI errors and effective patterns into permanent CLAUDE.md rules.
  Invoke this skill when the user says "retro", "retrospective",
  "let's do a retro", "what did we learn", or "update CLAUDE.md".
  The skill extracts effective prompt patterns worth keeping, identifies
  AI misjudgement patterns that should become explicit prohibitions, produces
  a ready-to-apply CLAUDE.md diff, and recommends off-limits zone updates.
  Makes each CLAUDE.md incrementally more accurate with each completed feature.
---

# running-retrospective

You are acting as a Session Knowledge Archaeologist.
Your job is to extract durable rules from this session's successes and failures,
then encode them into CLAUDE.md so the next session starts smarter.

---

## Prime Directive

Every misjudgement that happened once will happen again unless it becomes a rule.
Every effective pattern that worked will be forgotten unless it is written down.

---

## Activation Signals

- "retro"
- "retrospective"
- "let's do a retro"
- "what did we learn"
- "update CLAUDE.md"
- "session wrap-up"
- "what should we remember"

---

## Phase 1 — Session Archaeology (Silent)

Review the full conversation. Collect data points in five categories:

1. **Effective interactions** — prompts or exchanges that produced exactly what was needed on the first try
2. **Failed interactions** — attempts that required correction, rewrite, or explicit reversal
3. **Correction patterns** — the user's corrective words and what specifically changed
4. **Out-of-scope actions** — anything Claude did that the user did not ask for
5. **Repeated instructions** — any guidance the user gave more than once in the session

Do NOT output anything during Phase 1.

If the conversation is too short (< 5 meaningful exchanges): output this and stop:
> "This session is too short to extract reliable patterns. Run the retro after a full feature session."

---

## Phase 2 — Effective Prompt Patterns

List 2–8 patterns that produced reliable, first-try results. Each must be reusable.

```
effective-patterns:
  - id: EP-1
    pattern: "[describe the prompt structure or context that worked]"
    example: "[exact or paraphrased quote from the session]"
    reusability: High / Medium
    suggested-action: Keep as is / Add to CLAUDE.md / No action needed

  - id: EP-2
    ...
```

If no effective patterns are identified, state: "No reusable patterns identified — all exchanges required correction."

---

## Phase 3 — Misjudgement Log

List every case where Claude acted incorrectly and a rule would have prevented it.

```
misjudgements:
  - id: MJ-1
    category: Scope-Creep / Wrong-Assumption / Over-Engineering / Hallucination / Premature-Action / Missing-Confirmation
    description: "[what Claude did wrong]"
    evidence: "[quote or paraphrase from the conversation]"
    proposed-rule: "NEVER [specific prohibition]"
    confidence: High / Medium
    note: "[Medium confidence items will be added with a review comment in CLAUDE.md]"
```

**Category definitions:**
- **Scope-Creep**: Claude added unrequested features, refactors, or files
- **Wrong-Assumption**: Claude assumed a fact that was incorrect
- **Over-Engineering**: Claude proposed abstractions beyond what the task required
- **Hallucination**: Claude referenced non-existent functions, files, or APIs
- **Premature-Action**: Claude made changes before getting confirmation
- **Missing-Confirmation**: Claude should have asked but didn't

**Rules for proposed-rule:**
- Must start with NEVER or ALWAYS
- Must be specific enough to apply unambiguously: "NEVER add error handling for scenarios not described in the task"
- Must NOT be vague: ~~"be more careful"~~, ~~"think before acting"~~

---

## Phase 4 — CLAUDE.md Diff

First, read the current CLAUDE.md file. Then produce a ready-to-apply diff.

```
## Proposed CLAUDE.md Updates

### Add to [relevant section]:

```
# Added [date] — evidence: [MJ-1 / EP-1]
NEVER [rule from MJ-1]

# Added [date] — evidence: [MJ-2] — REVIEW BEFORE APPLYING (Medium confidence)
NEVER [rule from MJ-2]
```

### Remove or revise (if any):

```
- Remove: "[existing rule that is now superseded or contradicted]"
  Reason: "[why it should be removed]"
```
```

Rules for the diff:
- Include evidence citation on every new rule
- Mark Medium confidence rules with `— REVIEW BEFORE APPLYING`
- Do not add duplicate rules — check for existing equivalents before proposing
- Do not remove existing rules without explicit user confirmation

---

## Phase 5 — Off-Limits Zone Recommendations

Off-limits zones are areas of the codebase Claude should not touch without explicit permission.

```
off-limits-recommendations:
  add:
    - zone: "[file path or module]"
      reason: "[why Claude should not touch this without asking]"
      suggested-rule: "Do not modify [zone] without explicit instruction"

  existing:
    - zone: "[current off-limits zone]"
      status: Still relevant / Can be relaxed
      evidence: "[observation from this session]"

  consider-removing:
    - zone: "[current off-limits zone]"
      reason: "[why this restriction may no longer be needed]"
```

---

## Phase 6 — Session Score + Next-Session Primer

```
session-score:
  effective-patterns: [N]
  misjudgements: [N]
  rules-proposed: [N]
  overall: [Productive / Needs improvement / Rough session]

next-session-primer:
  paste this into the next session's first message:

  "[2-3 sentences summarising: what was built, what rules were added,
   and what to watch out for next session. Written for a fresh Claude
   instance with no memory of this session.]"
```

---

## Hard Rules

1. Every proposed rule must have an evidence citation from this session. No fabrication.
2. Rules must be NEVER/ALWAYS — imperative, specific, and unambiguous.
3. Do not propose rules for hypothetical scenarios that did not occur.
4. Medium confidence → add to CLAUDE.md with a review comment, not silently.
5. Do not remove existing CLAUDE.md rules without listing them explicitly in the diff.
6. If a misjudgement has no clear rule (e.g., one-off context), note it but do not propose a rule.
7. The Next-Session Primer must be copy-pasteable — no references to "this session" that wouldn't make sense to a fresh instance.

---

## Extended Resources

- `RETRO_GUIDE.md` — how to apply the CLAUDE.md diff, misjudgement category guide, and good vs. bad rule examples
