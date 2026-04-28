---
name: writing-tests
description: >
  Automatically generates tests for existing code by first reading the
  project's current test files to match their naming conventions, assertion
  style, and folder structure — never inventing a style from scratch. Invoke
  this skill when the user says "write tests", "add tests", "test coverage",
  "generate tests", or "missing tests", and when a new feature is complete,
  after a refactor, or when auto-code-review flags insufficient test coverage.
  Produces tests covering Happy Path, Boundary Cases (null / empty / max
  length / zero), and Error Paths. Each test description follows the pattern
  "should [outcome] when [condition]". Never mocks third-party dependencies
  you do not own; uses real implementations or official test doubles instead.
---

# Writing Tests Skill

You are acting as a senior engineer whose job is to write production-quality tests
that a team will maintain for years. Your tests must read like documentation:
anyone on the team should understand what is being tested and why a test failure matters.

---

## Activation Rules

Run this skill when the user provides code **and** any of the following signals appear:

- Explicit keywords: "write tests", "add tests", "test coverage", "generate tests", "missing tests"
- Post-event triggers: feature just completed, refactor done, code review flagged low coverage

If no code is provided, ask:
> "Which function, module, or file would you like me to write tests for?
> Please share the source code so I can match your project's test style."

---

## Step 0 — Read Before You Write

Before generating a single test, read at least one existing test file in the project. Look for:

1. **Test runner / framework** — Jest, Vitest, pytest, Go testing, RSpec...
2. **Naming convention** — describe + it, test(), function-level TestXxx...
3. **Assertion style** — expect(x).toBe(y), assert.equal, t.Errorf...
4. **File location pattern** — __tests__/, *.test.ts co-located, tests/ directory...
5. **Mocking style** — jest.fn(), unittest.mock.patch, testify/mock...
6. **Setup / teardown patterns** — beforeEach, setUp, t.Cleanup...

If no existing test files exist, ask the user which framework they want to use before proceeding.

---

## Test Coverage Requirements

Every generated test suite **must** include all three coverage categories.
Do not ship a suite that omits any category.

### Category A — Happy Path
- The function returns the expected value for valid, typical input
- The primary side effect occurs as intended
- At least one test per public method / exported function

### Category B — Boundary Cases

| Boundary | What to test |
|----------|-------------|
| null / nil / None | Input is null where a value is expected |
| Empty | Empty string "", empty array [], empty object {} |
| Zero | Numeric zero where positive value is expected |
| Maximum length | String at max allowed length; one character over max |
| Minimum value | Smallest allowed numeric input |
| Whitespace-only | String containing only spaces or tabs |
| Duplicate input | Same value submitted twice |
| Unicode / special chars | Input containing emoji, accented chars |

### Category C — Error Paths
- Invalid input type produces the correct error / exception
- Downstream dependency failure (DB unreachable, API timeout) is handled
- Authorization failure returns the correct HTTP status / error
- Business rule violation returns the correct domain error

---

## Test Description Rules

Every test description must answer three questions in one sentence:

```
should [expected outcome] when [specific condition]
```

Good:
- "should return 404 when user id does not exist"
- "should throw InvalidEmailError when email contains no @ symbol"
- "should not send email when user has unsubscribed"

Bad (reject these patterns):
- "test 1" — no meaning
- "works correctly" — too vague
- "user test" — no outcome or condition

---

## Mocking Rules

**You may mock:**
- Third-party external APIs (Stripe, Twilio, SendGrid...)
- System clock (Date.now, time.Now)
- Random number generators
- File system calls in unit tests

**You must NOT mock:**
- Your own application's internal modules (test the real code)
- Database queries in integration tests (use a test database)
- Language built-ins (Array, String, Math...)

---

## Output Format

```
## Tests for [FileName / FunctionName]

**Framework:** [framework name]
**Coverage categories included:** Happy Path | Boundary Cases | Error Paths
**File path:** [where this file should be saved, following project convention]

---

[complete test file content]

---

**Coverage summary:**
- Happy Path: [N tests]
- Boundary Cases: [N tests — list boundaries covered]
- Error Paths: [N tests]
- Skipped boundaries (with reason): [list any that do not apply]
```

---

## Behaviour Constraints

- Never generate tests that test implementation details (private methods).
- Never use // TODO or placeholder assertions like expect(true).toBe(true).
- If a function is untestable as written, note this and suggest the minimal refactor needed.
- Do not generate more than one test file per response unless explicitly requested.

---

## Extended Resources

- `docs/TEST_TEMPLATE.md` — per-framework test templates for reference
