# writing-tests

Test Generator — reads your existing test conventions before writing a single line.

## When to Use

Say any of the following to activate:
- "write tests"
- "add tests"
- "test coverage"
- "generate tests"
- "missing tests"

Also activates after a feature is complete, after a refactor, or when `auto-code-review` flags insufficient test coverage.

## What It Does

**Step 0 (mandatory):** Reads existing test files to detect framework, naming convention, assertion style, and folder structure. Never invents a style from scratch.

Then generates tests covering three mandatory categories:

| Category | What's Covered |
|----------|---------------|
| Happy Path | Valid input → expected output; primary side effects |
| Boundary Cases | null, empty, zero, max length, whitespace, unicode, duplicates |
| Error Paths | Invalid types, downstream failures, auth errors, business rule violations |

## Test Description Rule

Every test description must follow:
```
should [expected outcome] when [specific condition]
```

Good: `"should return 404 when user id does not exist"`
Bad: `"test 1"`, `"works correctly"`

## Mocking Rules

| May Mock | Must NOT Mock |
|----------|--------------|
| Third-party APIs (Stripe, Twilio) | Your own internal modules |
| System clock | Language built-ins |
| Random number generators | DB queries in integration tests |
| File system (unit tests only) | |

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill instructions |
| `docs/TEST_TEMPLATE.md` | Templates for Jest, pytest, Go, and RSpec |
