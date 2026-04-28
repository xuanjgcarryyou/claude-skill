# refactoring-safely

Pre-Refactor Contract — snapshots the public API and waits for approval before touching any code.

## When to Use

Say any of the following to activate:
- "refactor"
- "clean up" / "cleanup"
- "restructure"
- "reorganize"
- "simplify the code"
- "extract this into"
- "split this module"

## What It Does

**Phase 1 (silent):** Reads the target file and scans the codebase for all consumers.

**Phase 2 (outputs):** Five mandatory sections:
1. **Public API Snapshot** — every exported function signature and type
2. **Consumer Map** — every file that imports from the target module
3. **Invariants Declaration** — behavioral contracts derived from tests/docs
4. **Existing Test Coverage** — which tests cover the target
5. **Risk Register** — HIGH / MEDIUM / LOW risk items

**Phase 3 (gate):** Outputs an approval prompt and stops. No code is changed.

**Phase 4 (after APPROVED):** Refactors one logical unit at a time, showing a diff after each change.

## Approval Protocol

- Type `APPROVED` to begin refactoring (non-breaking changes)
- Type `APPROVED BREAKING` to confirm API-breaking changes
- "just do it" or "go ahead" without the keyword → skill declines and repeats the prompt

## Hard Rules

- No code changes before snapshot is approved
- Breaking changes mid-refactor require a second APPROVED BREAKING
- One diff per logical unit — no batching
- Missing tests on the refactor target → recommends writing tests first
