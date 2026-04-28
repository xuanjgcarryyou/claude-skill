# summarizing-pr

PR Description Specialist — six fixed sections, ready to paste into GitHub.

## When to Use

Say any of the following to activate:
- "write PR description"
- "summarize my changes"
- "create PR"
- "PR ready"
- "write the PR"
- "generate PR"
- "draft the pull request"

## What It Does

Produces a complete PR description with six sections in a fixed order:

| Section | Purpose |
|---------|---------|
| 1 — What Changed | Specific bullet list of changes with file/function names |
| 2 — Why | Motivation paragraph + issue/ticket link |
| 3 — How to Test | Numbered steps with expected results |
| 4 — Risks & Edge Cases | Markdown table with Severity and Mitigation |
| 5 — Rollback | Exact rollback commands + estimated time + data impact |
| 6 — Checklist | Unchecked `- [ ]` items for the author to verify |

After outputting, runs a **quality self-check** and appends `NEEDS CLARIFICATION` if any section is incomplete.

## Hard Rules

- Forbidden phrases: "fixed stuff", "various improvements", "minor tweaks" — triggers a rewrite of that bullet
- Rollback section is mandatory on every PR, even small ones
- Checklist items are never pre-checked — the author checks them, not Claude
- "write something quick" → still outputs all six sections; missing info marked `[needs clarification]`

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill instructions |
| `PR_TEMPLATE.md` | Blank template with section-by-section writing guide |
