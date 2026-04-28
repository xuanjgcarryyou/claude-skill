# Retrospective Guide

Reference for engineers applying retrospective output and writing high-quality CLAUDE.md rules.

---

## How to Apply the CLAUDE.md Diff

1. Open `CLAUDE.md` in the project root.
2. Locate the section the diff targets (e.g., "Behavioral Rules", "Off-Limits Zones").
3. Copy the proposed rule exactly as written, including the evidence comment.
4. For **Medium confidence** items (marked `— REVIEW BEFORE APPLYING`):
   - Read the evidence citation before adding.
   - If the scenario was genuinely a one-off, skip the rule.
   - If the pattern is likely to recur, add it.
5. For **High confidence** items: add without further review.
6. After applying, re-read the full CLAUDE.md to check for duplicates or contradictions.

---

## Misjudgement Category Guide

Use these definitions when classifying an AI misjudgement in Phase 3.

| Category | Definition | Example |
|----------|-----------|---------|
| **Scope-Creep** | Claude added unrequested features, files, or refactors | "I also refactored the related module while I was there" |
| **Wrong-Assumption** | Claude assumed a fact that was not stated and was incorrect | Assumed the DB was PostgreSQL when it was SQLite |
| **Over-Engineering** | Claude proposed abstractions beyond what the task required | Added a factory pattern for a function called once |
| **Hallucination** | Claude referenced non-existent functions, files, or APIs | Suggested using `Array.prototype.flatDeep()` which does not exist |
| **Premature-Action** | Claude made changes before getting confirmation | Deleted a file after user said "we probably don't need this" |
| **Missing-Confirmation** | Claude should have asked but proceeded | Chose a database schema without asking about existing constraints |

---

## Good Rule vs. Bad Rule Examples

Rules must be **imperative, specific, and unambiguous**.

### Bad Rules (reject these)

| Rule | Why it fails |
|------|-------------|
| "Be more careful with file deletions" | Not actionable — "more careful" is undefined |
| "Think before making changes" | Cannot be operationalized — Claude always "thinks" |
| "Avoid scope creep" | Too vague — what counts as scope? |
| "Don't assume things" | True but untargeted — every action involves assumptions |

### Good Rules (accept these)

| Rule | Why it works |
|------|-------------|
| `NEVER delete a file unless the user's message contains the word "delete"` | Specific trigger and condition |
| `NEVER add a new npm package without running the auditing-dependencies skill first` | Clear action, clear gate |
| `ALWAYS ask which database the project uses before generating any SQL` | Specific when, specific what |
| `NEVER refactor code outside the file explicitly named in the user's message` | Named condition, named boundary |

---

## Rule Writing Formula

```
[NEVER / ALWAYS] [specific action] [condition / context]
```

Examples:
- `NEVER add console.log statements to production code`
- `ALWAYS include a rollback step in any migration output`
- `NEVER suggest renaming a public API symbol without first mapping its consumers`
- `ALWAYS confirm the test framework before generating test files`

---

## Next-Session Primer — Writing Guide

The primer is 2–3 sentences copied into the **first message** of the next session.
It must make sense to a fresh Claude with no memory of this session.

### Good Primer

> "We are building a user authentication flow for a Next.js app using Prisma + PostgreSQL.
> A retro on 2026-04-28 added two rules: never add dependencies without auditing-dependencies,
> and always ask for the target file before refactoring. The current task is implementing
> the password reset email flow — the API contract is locked in `.claude/skills/contracting-api/`."

### Bad Primer

> "Continue where we left off. Remember the things we discussed."
> — Useless to a fresh Claude instance with no context.

---

## Off-Limits Zone Reference

Off-limits zones belong in CLAUDE.md under a dedicated section:

```markdown
## Off-Limits Zones

The following files/directories must not be modified without explicit user instruction:

- `src/auth/` — production auth logic; touch only with security review
- `prisma/migrations/` — never hand-edit; use migrating-schema skill
- `public/assets/` — managed by design team; do not touch
```

When adding a zone, always state the reason. A zone without a reason will be ignored or forgotten.
