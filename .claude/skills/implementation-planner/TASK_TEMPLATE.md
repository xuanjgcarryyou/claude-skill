# Task Package Template

A reusable template for a single implementation task. Every task package produced by the implementation-planner skill must follow this format exactly.

The goal: a subagent with no prior context should be able to pick up this package, open the referenced files, and execute the task without asking a single follow-up question.

---

## How to Fill Each Field

**Task [N]: [Short Title]**
- N = sequential task number across all phases
- Title = specific enough to distinguish from other tasks (e.g., "User Authentication Middleware" not "Auth")

**Phase:** 1 / 2 / 3 / 4
- 1 = Foundation, 2 = Core, 3 = Integration, 4 = Secondary

**Can run parallel with:** [task IDs] or "none"
- List task IDs in the same phase that have no shared dependencies and can be worked on simultaneously
- Be honest — only mark as parallel if there's truly no shared state or interface conflict

**Depends on:** [task IDs] or "none"
- Tasks that must be complete before this one starts
- Include both direct dependencies (uses their output) and soft dependencies (needs their interface defined)

**Objective:**
- Exactly one sentence — what this task achieves when done
- Write it as a completion statement: "Implement X that does Y" or "Create X so that Z is possible"

**Scope:**
- What is included: bullet list of specific things this task covers
- What is explicitly NOT included: things that might seem related but belong to another task
- Scope boundaries prevent subagents from over-building or stepping on other tasks

**Input (what must exist before starting):**
- Specific artifacts that must be complete: schema files, interfaces, modules, environment variables
- Be concrete — "DB schema defined in /db/schema.sql" not just "database ready"
- If an interface must exist but the implementation doesn't have to be complete, say so

**Output (what this task produces):**
- Specific files, classes, functions, endpoints, or data this task creates
- List the exact names or paths if known
- This is the artifact that other tasks will depend on

**Acceptance Criteria:**
- Concrete, observable conditions that define "done" for this task
- Each criterion must be testable without knowing the implementation details
- Examples: "API returns 200 for valid input and 400 for missing fields", "DB insert is idempotent on duplicate key", "function handles null input without throwing"
- Do NOT write vague criteria like "works correctly" — every criterion must be verifiable

**Implementation Method:**
- Chosen approach: specific pattern, class structure, or integration method
- Key library/tool: exact package name and version if relevant
- Key patterns to follow: e.g., "follow repository pattern used in /services/", "match error handling in /middleware/errorHandler.ts"
- Why this method: one sentence rationale (not obvious things — only decisions worth explaining)

**Interface Contract:**
- The public surface this task exposes to other tasks
- Include: function signatures, API endpoint paths + methods + request/response shapes, event names + payloads, or exported class/interface names
- This is what other tasks will code against — it must be precise
- If the contract isn't fully known yet, note that as a risk

**Constraints:**
- Must follow [specific naming convention, error handling style, project pattern]
- Must not [introduce new dependencies without flagging, bypass existing middleware, etc.]
- Must be consistent with [specific files or patterns in the project]

**Security Constraints:**
- Are there user-controlled inputs fed to DB queries, shell commands, or HTML output? If yes, specify how they must be sanitized or parameterized.
- Does this task handle credentials, tokens, or secrets? If yes, state explicitly: where they come from (env var, secret store) and that they must never be logged or hardcoded.
- Does this task gate access to resources? If yes, specify which auth/permission checks are required and where they must be enforced.
- If none of the above apply, write: "No security-sensitive inputs or operations in this task."

**Error Handling:**
- Specify the error propagation strategy: throw exceptions / return error values / emit events
- Specify log level and format for errors in this task (e.g., `console.error('[MODULE] message', err)`)
- Specify what happens on partial failure — does this task roll back, retry, or fail fast?
- Must be consistent with the error handling convention in [reference file or module]

**Context (for subagent):**
- Copy-paste the relevant spec excerpts, schema snippets, or interface definitions the subagent needs
- Do not assume the subagent has read the architecture spec — include what they need here
- Keep this focused — only include what's actually needed for this task

**Risks / Unknowns:**
- What might be harder than expected (third-party API behavior, edge cases in the domain logic, etc.)
- What decisions were deferred to the implementer (things the plan didn't prescribe)
- What to do if X fails or isn't available (fallback decisions, if any)

**Rollback / On Failure:**
- If this task produces side effects (DB writes, external API calls, file writes), specify how to undo or recover if it fails midway
- For idempotent tasks: state that explicitly ("safe to re-run — all operations are idempotent")
- For destructive or irreversible operations: flag clearly and specify what must be backed up or verified before starting

---

## Template

```
## Task [N]: [Short Title]

**Phase:** [1 / 2 / 3 / 4]
**Can run parallel with:** [task IDs or "none"]
**Depends on:** [task IDs or "none"]

**Objective:**
[One sentence — what this task achieves.]

**Scope:**
Included:
- [specific item]
- [specific item]

Not included:
- [specific item — belongs to Task X]
- [specific item — out of scope for v1]

**Input (what must exist before starting):**
- [artifact or state that must be complete]
- [artifact or state that must be complete]

**Output (what this task produces):**
- [file or artifact]
- [file or artifact]

**Acceptance Criteria:**
- [Observable, testable condition 1]
- [Observable, testable condition 2]

**Implementation Method:**
- Chosen approach: [specific method or pattern]
- Key library/tool: [package name]
- Key patterns to follow: [reference to existing code or convention]
- Why: [one sentence — only if the choice isn't obvious]

**Interface Contract:**
[Function signatures, endpoint definitions, event names, or exported types this task exposes]

**Constraints:**
- [Must / must not statement]
- [Must / must not statement]

**Security Constraints:**
- [User inputs / SQL / auth / secrets handling — or "No security-sensitive inputs or operations in this task."]

**Error Handling:**
- [Throw / return / emit strategy, log format, partial failure behavior]

**Context (for subagent):**
[Relevant spec excerpts, schema snippets, or interface definitions copied here]

**Risks / Unknowns:**
- [What might be harder than expected]
- [Deferred decision for the implementer]

**Rollback / On Failure:**
- [Idempotent / reversible / irreversible — and what to do if it fails]
```

---

## Checklist Before Finalizing a Task Package

Before including a task package in the final plan, verify:

- [ ] A subagent with no prior context could start this task immediately
- [ ] The scope boundaries are clear — no ambiguity about what's in or out
- [ ] All dependencies are listed (nothing assumed to "just be there")
- [ ] The interface contract is specific enough for other tasks to code against
- [ ] Acceptance Criteria are observable and testable — no vague "works correctly"
- [ ] Security Constraints explicitly address inputs, credentials, and auth (or declare none apply)
- [ ] Error Handling strategy matches the project convention and is stated explicitly
- [ ] Rollback / On Failure behavior is specified for any side-effectful task
- [ ] Context section contains everything the subagent actually needs
- [ ] Risks / Unknowns are honest — nothing swept under the rug
- [ ] Parallel tasks are correctly identified (same phase, no shared mutable state)
