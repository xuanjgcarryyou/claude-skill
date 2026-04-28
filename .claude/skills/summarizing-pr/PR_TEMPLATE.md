# PR Description Template

Blank template for manually writing a PR description, or for reference when reviewing Claude's output.
Fill in every section — do not delete any section, even for small changes.

---

## What Changed

<!--
List specific changes as bullets. Name the file, function, or behaviour that changed.
Bad:  "Fixed a bug"
Good: "Fixed null pointer in `UserService.authenticate()` when email field is missing"

Each bullet should answer: what was changed and where?
-->

-
-
-

---

## Why

<!--
One paragraph: what problem does this solve, or what requirement does this fulfil?
Always include a link to the related issue or ticket.
-->

Resolves: #

---

## How to Test

<!--
Numbered steps. Each step must have an expected result.
Include any env vars, seed data, or test accounts needed.
-->

1.  → Expected:
2.  → Expected:
3. Run `[test command]` → Expected: all tests pass

**Setup required:**
- Env vars:
- Seed data / test accounts:

---

## Risks & Edge Cases

<!--
At minimum one row. If no risks, explain why:
"Low: This change only affects [scope]. Existing tests cover the affected paths."
-->

| Risk | Severity | Mitigation |
|------|----------|-----------|
|  | High / Medium / Low |  |

---

## Rollback

<!--
Always include, even for small changes.
If a DB migration is involved, include the down migration command.
-->

```bash
# Rollback command:

```

Estimated rollback time:
Data impact: None / Reversible / Requires manual fix

---

## Checklist

<!--
Do NOT pre-check these items. The author checks them before merging.
Add items specific to this PR's risk profile.
-->

- [ ] Tests added or updated
- [ ] No secrets or credentials committed
- [ ] Linting passes (`[command]`)
- [ ] No breaking API changes (or breaking changes are documented above)
- [ ] PR description reviewed for accuracy before requesting review

---

## Section Writing Guide

### What Changed — Good vs. Bad

| Bad | Good |
|-----|------|
| "Fixed auth bug" | "Fixed missing null check in `AuthService.validate()` that caused 500 on empty token" |
| "Updated styles" | "Updated `Button` component background from `#333` to `#1a1a1a` for WCAG AA contrast" |
| "Various improvements" | Never acceptable — list every change individually |

### Risks — Severity Guide

| Severity | Meaning |
|----------|---------|
| High | Could cause data loss, outage, or security exposure |
| Medium | Could break a specific user flow or cause incorrect data |
| Low | Cosmetic, test-only, or isolated to non-critical path |

### Checklist — What to Add

Add custom checklist items when the PR involves:
- DB migration → `- [ ] Migration down script tested locally`
- External API → `- [ ] Verified against staging API`
- Auth change → `- [ ] Tested with expired/invalid token`
- File upload → `- [ ] Tested with oversized and malformed file`
- New dependency → `- [ ] Dependency audit completed`
