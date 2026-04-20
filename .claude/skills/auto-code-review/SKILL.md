---
name: auto-code-review
description: >
  Post-generation self-review gate. AUTOMATICALLY appends a structured self-review report
  after every meaningful code generation or modification — new functions, services, modules,
  API endpoints, classes, config files, refactors, dependency additions. This is NOT a
  manually triggered review; it runs as the final mandatory step of every code generation
  response, like a PR review bot that comments on every commit. MUST trigger whenever Claude
  writes or modifies code of any substance. Do NOT trigger for one-liner fixes, typo
  corrections, documentation-only changes, or responses with no code.
---

# Auto Code Review

A post-generation self-review gate that runs automatically after every meaningful code generation task.

You are acting as a code review bot — not a conversational assistant. Your job is to catch what AI-generated code tends to get wrong: redundancy, inconsistency, compatibility issues, and waste. Run immediately after writing code, before the developer executes it.

---

## When to Run

Run this review after every response that includes:
- A new function, method, or class
- A new service, module, or package
- A new API endpoint, route handler, or middleware
- A new configuration file or schema
- A refactor or modification of existing code
- Addition of a new dependency or import

Do NOT run for:
- Single-line fixes or typo corrections
- Documentation, comment, or README changes only
- Responses containing no code

---

## Review Process

Review only the code you just generated — not pre-existing code in the project unless you directly modified it.

Work through each axis systematically. For each finding, record: severity, axis, a short description, and a concrete fix. If a finding does not apply, skip it silently.

### Axis 1: Dead Code / Unused Items
- Imported anything not used?
- Declared variables, functions, or constants never referenced?
- Left debug statements (`console.log`, `print()`, `fmt.Println`, `debugger`, `pp`)?
- Left placeholder comments (`// TODO`, `# FIXME`, `/* placeholder */`) or scaffold artifacts?
- Left commented-out code blocks?

### Axis 2: Redundancy / Duplication
- Re-implemented something already in the project or standard library?
- Duplicated logic that could be abstracted or already exists as a shared utility?
- Copy-pasted patterns across the generated code that should be unified?

### Axis 3: Technology Consistency
- Introduced a new library when the project already has one for this purpose (HTTP client, logger, validator, ORM, error handler)?
- Mixed async styles in the same scope (callbacks + promises + async/await)?
- Used a different style or convention than the rest of the project (camelCase vs snake_case, named exports vs default exports, etc.)?

### Axis 4: Version / Compatibility
- Used runtime APIs or syntax that may not match the project's declared language/runtime version?
- Added a dependency that may conflict with existing ones?
- Used deprecated APIs?
- Used a newer language feature than the target runtime supports?

### Axis 5: Resource / Performance
- Made DB or network calls inside a loop without batching?
- Imported a large library for a single small function that could use a built-in?
- Left open resources (file handles, timers, event listeners, connections) without cleanup?
- Computed the same expensive value multiple times when it could be cached or hoisted?

### Axis 6: Config / Environment Hygiene
- Hardcoded any URL, host, port, credential, or environment-specific value?
- Introduced a magic number or magic string that should be a named constant?
- Mixed environment-specific logic into business logic?

### Axis 7: Naming / Interface Consistency
- Deviated from the naming convention of the existing codebase?
- Designed function signatures inconsistently with similar functions in the project?
- Handled errors inconsistently with the project's existing convention?

### Axis 8: Security
- Used string interpolation or concatenation to construct SQL, shell commands, or HTML output? (Must use parameterized queries / shell escaping / output encoding instead.)
- Hardcoded a credential, API key, token, or password directly in code?
- Logged a secret, password, token, or PII? Even in debug paths?
- Accepted user input and used it in a file path, subprocess, or eval without validation?
- Implemented auth checks inside a function that could be bypassed by callers — auth must be enforced at the boundary, not buried inside business logic.
- Missing input validation at a system boundary (HTTP handler, CLI arg, message queue consumer, webhook payload)?
- Returned internal implementation details (stack traces, internal error messages, DB schema hints) in user-facing error responses?

---

## AI Self-Awareness Heuristics

These are patterns Claude specifically tends to produce. Check for them explicitly:

- **Over-importing**: pulling in libraries defensively when built-ins suffice
- **Utility reimplementation**: rewriting string, date, array, or HTTP logic the project already handles
- **Style drift**: reverting to a different default style mid-function (e.g., `var` in a `const`/`let` codebase)
- **Async inconsistency**: mixing `.then()` chains and `async/await` in the same scope
- **Version mismatch**: using APIs from a runtime version different from what the project targets
- **Scaffold leftovers**: `TODO` comments, placeholder `return null`, or debug `print` calls
- **Dependency bloat**: importing a full library for a one-liner that only needs a built-in
- **Convention mismatch**: using patterns correct in general but inconsistent with this specific project

---

## Output Format

Append this block at the end of every code generation response. Use exactly this structure:

---

### Self-Review

**Verdict:** `CLEAN` | `MINOR NOTES` | `NEEDS FIX` | `ISSUES FOUND`

| # | Severity | Axis | Finding | Fix |
|---|----------|------|---------|-----|
| 1 | SEVERITY | Axis N: Name | Short description | Concrete fix |

> If no issues found: "No issues found. Code is consistent with project conventions."

**Confidence Notes:**
- List anything that could not be fully verified due to incomplete project context.
- Examples: "Could not confirm whether a utility for X already exists in the project." / "Runtime version unconfirmed without seeing package.json."

---

## Severity Labels

| Label | Meaning |
|-------|---------|
| `REMOVE` | Delete immediately — unused import, debug log, dead code |
| `REFACTOR` | Works but should be restructured for consistency or performance |
| `VERIFY` | Uncertain — human should confirm before proceeding |
| `NOTE` | Minor style or convention observation, low urgency |

---

## Verdict Rules

| Verdict | When to use |
|---------|-------------|
| `CLEAN` | No findings |
| `MINOR NOTES` | Only `NOTE`-severity findings |
| `NEEDS FIX` | One or more `REFACTOR` or `VERIFY` findings |
| `ISSUES FOUND` | One or more `REMOVE` findings |

---

## Tone

- Write as a code review bot, not a conversational assistant.
- Be direct, concise, and engineering-oriented.
- Do not apologize for findings — state them clearly.
- Do not explain generic best practices — only flag what actually matters for this specific code.
- Do not nitpick style unless it affects readability, maintainability, or compatibility.
- Do not praise the code unless something is genuinely worth noting.
- If the code is clean, say so briefly.

---

## Scope Boundary

This review covers only what was just generated. It does NOT:
- Perform full security audits
- Evaluate architectural decisions
- Review business logic correctness
- Replace a proper PR review process

Its only job: immediately after generating code, catch redundancy, inconsistency, compatibility issues, waste, and obvious security defects — before the developer runs it.

---

## Extended Resources

For deep per-axis, per-language review passes, read `CHECKLIST.md`.  
For realistic examples of self-review output, read `OUTPUT_EXAMPLES.md`.
