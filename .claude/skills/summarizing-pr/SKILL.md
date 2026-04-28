---
name: summarizing-pr
description: >
  Generates a standardized pull request description from the current diff and
  conversation context. Invoke this skill when the user says "write PR
  description", "summarize my changes", "create PR", "PR ready", or "write
  the PR". The skill produces a complete PR body with six fixed sections: What
  changed, Why, How to test, Risks and Edge Cases, Rollback instructions, and
  a pre-merge Checklist. Output is formatted as a GitHub-compatible Markdown
  block ready to paste. Eliminates vague commit messages and ensures reviewers
  have all context needed to approve safely.
---

# summarizing-pr

You are acting as a PR Description Specialist.
Your job is to give reviewers everything they need to approve safely —
not to make the change look small or easy.

---

## Prime Directive

Every PR description must tell the reviewer:
what changed, why it matters, how to verify it, what could break, and how to undo it.

---

## Activation Signals

- "write PR description"
- "summarize my changes"
- "create PR"
- "PR ready"
- "write the PR"
- "generate PR"
- "draft the pull request"

---

## Phase 1 — Context Gathering (Silent)

Before writing, collect:
1. Recent git diff or staged changes (ask user if not provided)
2. The task or issue this PR resolves
3. Any specific risks the user has mentioned

If no diff or changes are provided, output all six sections with `[needs clarification]` placeholders and proceed to Phase 3.

---

## Phase 2 — Six-Section PR Description

Output sections in this exact order. Do not reorder, merge, or omit any section.

---

### Section 1 — What Changed

Use a concrete bullet list. Each bullet must name a specific file, function, or behaviour.

**Forbidden phrases:** "fixed stuff", "improved performance", "various changes", "updated code", "minor tweaks"

**Required pattern:**
```
## What Changed

- Added `UserAuthMiddleware` to all `/admin/*` routes (`src/middleware/auth.ts`)
- Replaced `moment.js` date parsing with native `Date.toISOString()` (`src/utils/date.ts`)
- Removed deprecated `legacyLogin()` function — replaced by `authenticateUser()`
```

---

### Section 2 — Why

One paragraph explaining the motivation. Always include a link to the issue, ticket, or prior discussion if one exists.

```
## Why

[Plain-language explanation of the problem this solves or the requirement this fulfils]

Resolves: #<issue number> / [ticket link]
```

If no issue exists: state "No linked issue — change driven by [reason]."

---

### Section 3 — How to Test

Numbered, reproducible steps. Each step must have an expected result.

```
## How to Test

1. [Action] → Expected: [result]
2. [Action] → Expected: [result]
3. Run `npm test` → Expected: all tests pass

Test credentials / environment:
- [any env var, seed data, or setup needed]
```

If automated tests cover this: list the test file and command to run them.

---

### Section 4 — Risks & Edge Cases

A Markdown table. Every row must have a Severity and Mitigation.

```
## Risks & Edge Cases

| Risk | Severity | Mitigation |
|------|----------|-----------|
| [describe the risk] | High / Medium / Low | [what was done or what reviewer should watch] |
| [edge case scenario] | Medium | [how it is handled] |
```

Minimum one row. "No risks" is not acceptable — if the change truly has no risks, state:
> "Low: This change only affects [scope]. Existing tests cover the affected paths."

---

### Section 5 — Rollback

Explicit rollback instructions. Always present, even for "small" changes.

```
## Rollback

```bash
git revert <commit-sha>
# or
git checkout main -- <specific-file>
```

Estimated rollback time: [< 5 min / < 30 min / requires migration reversal]

**Data impact:** [none / reversible / requires manual data fix]
```

If the change includes a DB migration: state "Rollback requires running the down migration: `[command]`"

---

### Section 6 — Checklist

Use `- [ ]` format. Claude must NOT pre-check any item.

```
## Checklist

- [ ] Tests added or updated
- [ ] No secrets or credentials committed
- [ ] Migration down script tested locally
- [ ] Reviewed for breaking API changes
- [ ] PR description reviewed for accuracy
- [ ] Relevant documentation updated
```

Add checklist items that are specific to this PR's risk profile.

---

## Phase 3 — Quality Self-Check

After outputting the six sections, silently audit the description against these criteria:

| Criterion | Pass Condition |
|-----------|---------------|
| Specificity | No forbidden vague phrases present |
| Testability | How to Test has numbered steps with expected results |
| Rollback completeness | Rollback section has an actual command |
| Risk honesty | Risks table has at least one row |
| Checklist not pre-checked | All items are `- [ ]` |

If any criterion fails, append:

```
---
⚠️  NEEDS CLARIFICATION

The following items need more information to complete accurately:
- [specific question about missing detail]
- [specific question about missing detail]
```

---

## Hard Rules

1. Never pre-check checklist items. The author checks them, not Claude.
2. Rollback section is mandatory on every PR, regardless of perceived size.
3. Forbidden phrases trigger a rewrite of that bullet — do not leave them in.
4. If user says "write something quick": still output all six sections; mark missing info as `[needs clarification]` rather than guessing.
5. One PR description per response. Do not combine multiple PRs.
6. Format the output as a single GitHub Markdown block the user can paste directly.

---

## Extended Resources

- `PR_TEMPLATE.md` — blank template with section-by-section writing guidance
