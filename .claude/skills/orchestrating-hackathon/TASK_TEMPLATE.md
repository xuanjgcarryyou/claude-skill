# TASK_[A/B/C].md — Stage 3 Task Package Template

> This document is a complete execution package for one teammate.
> A coding model must be able to implement this without asking foundational clarification questions.
> The orchestrator describes WHAT code needs to exist. It does not write the code.

---

## Owner

- **Teammate:** [A / B / C]
- **Branch Name:** `feat/[module-name]`
- **Owned Folders:** (list every folder this teammate exclusively owns)
- **Integration Branch:** `main` (or `dev` — match ARCHITECTURE.md)

---

## Mission

> One paragraph: what this task accomplishes and why it matters to the demo.
> Include the judging angle (e.g., "This module is what judges will interact with directly").

---

## Scope In

> What this task explicitly includes. Be specific — name files, endpoints, and features.

- [ ] Feature / file / endpoint 1
- [ ] Feature / file / endpoint 2
- [ ] Feature / file / endpoint 3

---

## Scope Out

> What this task explicitly does NOT include. Prevents scope creep and ownership confusion.

- Does NOT implement: ___
- Does NOT own: ___ (owned by Teammate ___)
- Does NOT integrate with: ___ (integration handled by Teammate ___)

---

## Dependencies

> What must exist before this task can start. Name the specific artifact or contract.

| Dependency | Source | Status |
|-----------|--------|--------|
| Contract: `[endpoint name]` | ARCHITECTURE.md §Interface Contracts | Locked |
| Schema: `[TypeName]` | ARCHITECTURE.md §Shared Schemas | Locked |
| DB connection string | `.env.local` | Must be set up first |

---

## Files To Create or Edit

> List every file this task touches. Use exact paths from the Directory Tree in ARCHITECTURE.md.

| File Path | Action | Purpose |
|-----------|--------|---------|
| `module-a/index.ts` | Create | Entry point |
| `module-a/service.ts` | Create | Core business logic |
| `shared/types.ts` | Edit | Add `[TypeName]` interface |

---

## Per-File Implementation Plan

> For each file, describe what must exist inside it. This is the most important section.
> The orchestrator defines WHAT the code must do — not HOW to write it line by line.

### `[file path]`

- **Purpose:** What this file is responsible for
- **Functions / Classes / Components to add:**
  - `functionName(param: Type): ReturnType` — what it does
    - Happy path: ___
    - Error path: ___ (what to throw / return / log)
  - `ClassName` — what it represents
    - Methods: `method1()`, `method2()`
- **Imports needed:** (what this file pulls from other modules)
- **Contract it must satisfy:** (which Interface Contract from ARCHITECTURE.md applies)
- **What can be stubbed first:** (what can be a placeholder before integration)

### `[next file path]`

- [Repeat the same structure]

---

## Logic Flow

> Describe the end-to-end flow for the primary use case this task handles.
> Written as numbered steps — concrete enough to guide implementation order.

1. User triggers [action] → [entry point] receives `{...}`
2. [Entry point] validates input, calls `[function]`
3. `[function]` reads from DB / calls external API / computes result
4. Result is returned as `{...}` to [caller]
5. [Caller] formats response and returns to user

---

## API / Contract Dependencies

> Copy the exact contracts from ARCHITECTURE.md that this task must implement or consume.
> Do not ask teammates — use these copied definitions as ground truth.

### Contracts This Task Implements (owns the server side)

```
Method: POST /api/[endpoint]
Request: { field: type }
Response (200): { field: type }
Error (4xx): { error: string }
```

### Contracts This Task Consumes (calls someone else's module)

```
Module B exposes: GET /api/[endpoint]
Expected response: { field: type }
```

---

## Mock Strategy

> What this task will fake during development and when it must be replaced with real code.

| Mocked Item | Mock Method | Must Be Real By |
|------------|-------------|----------------|
| [External API call] | Return hardcoded JSON | Demo day |
| [Auth token] | Hardcoded test token | End of Day 1 |
| [DB write] | In-memory array | After schema migration |

---

## Acceptance Criteria

> Observable conditions a person can verify without reading the code.
> Written as "Given / When / Then" or as checkboxes.

- [ ] Given [state], when [action], then [observable result]
- [ ] Given [state], when [action], then [observable result]
- [ ] Error case: Given [bad input], when [action], then [specific error message / status]

---

## Definition of Done

- [ ] All Scope In items implemented
- [ ] All acceptance criteria pass manually
- [ ] No TypeScript / lint errors
- [ ] Branch pushed and PR opened against integration branch
- [ ] At least one teammate can pull the branch and see it work

---

## Risks

| Risk | Likelihood | What To Do If It Happens |
|------|-----------|------------------------|
| [Contract from Teammate X changes] | Low | Re-read ARCHITECTURE.md, flag to orchestrator |
| [Third-party API rate limit hit] | Medium | Switch to mock immediately |
| [Feature takes longer than estimated] | Medium | Cut to Scope Out first, preserve demo narrative |

---

## Decision

> Why this task is scoped this way. Reference the architecture and ownership map.

---

## Rationale

> Why this teammate owns this module. What made the boundary clean.

---

## Rejected Options

| Option | Why Rejected |
|--------|-------------|
| [Alternative scope boundary] | |

---

## Open Questions

- [ ] Question that could affect implementation — escalate to orchestrator if unresolved

---

*Generated by `orchestrating-hackathon` — Stage 3 complete*
