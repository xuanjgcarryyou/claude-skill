# auto-code-review

A Claude Code skill that automatically appends a structured self-review report after every meaningful code generation task.

Designed for vibe-coding workflows where you describe what you want, Claude generates code, and Claude **immediately reviews what it just wrote** — without you having to ask.

---

## What It Does

After every response where Claude writes or modifies substantial code, this skill appends a self-review section structured as:

- A **verdict** (`CLEAN` / `MINOR NOTES` / `NEEDS FIX` / `ISSUES FOUND`)
- A **findings table** with severity, axis, description, and a concrete fix
- **Confidence notes** listing what could not be verified without more project context

The review checks 7 axes: dead code, redundancy, technology consistency, version compatibility, resource/performance, config hygiene, and naming consistency.

It is especially tuned for patterns that AI-generated code tends to produce: over-importing, utility reimplementation, style drift, async inconsistency, scaffold leftovers, and dependency bloat.

---

## Installation

Copy the `auto-code-review/` directory into your Claude Code skills folder:

```
~/.claude/skills/auto-code-review/
├── SKILL.md
├── CHECKLIST.md
├── OUTPUT_EXAMPLES.md
└── README.md
```

No configuration required. The skill is loaded automatically when Claude Code starts.

---

## Usage

No manual trigger needed.

After Claude finishes writing code, the self-review section appears automatically at the bottom of the response. Example:

```
[... generated code ...]

---

### Self-Review

**Verdict:** ISSUES FOUND

| # | Severity | Axis | Finding | Fix |
|---|----------|------|---------|-----|
| 1 | REMOVE | Axis 1: Dead Code | `axios` imported but never used | Remove import |
| 2 | REMOVE | Axis 6: Config Hygiene | API key hardcoded | Use process.env.API_KEY |

**Confidence Notes:**
- Could not confirm whether project already has a mailer utility.
```

---

## Trigger Conditions

The review runs automatically after:

| Triggers | Does NOT trigger |
|----------|-----------------|
| New function, method, or class | Single-line fix or typo correction |
| New service, module, or package | Documentation or comment changes only |
| New API endpoint or route handler | Responses with no code |
| New configuration file or schema | |
| Refactor or modification of existing code | |
| Addition of a new dependency | |

---

## Review Axes

| # | Axis | What It Catches |
|---|------|----------------|
| 1 | Dead Code / Unused Items | Unused imports, variables, debug logs, placeholders, commented-out code |
| 2 | Redundancy / Duplication | Reimplementing stdlib or project utilities, duplicated logic |
| 3 | Technology Consistency | Wrong HTTP client, logger, validator, ORM; mixed async styles |
| 4 | Version / Compatibility | APIs not available in declared runtime version, deprecated usage |
| 5 | Resource / Performance | N+1 queries, unclosed resources, expensive recomputation, dependency bloat |
| 6 | Config / Environment Hygiene | Hardcoded credentials, magic numbers/strings, env logic in business code |
| 7 | Naming / Interface Consistency | Naming conventions, function signature patterns, error handling style |

---

## Severity Labels

| Label | Meaning |
|-------|---------|
| `REMOVE` | Delete immediately — dead code, debug log, hardcoded secret |
| `REFACTOR` | Works but should be restructured for consistency or performance |
| `VERIFY` | Uncertain — confirm before proceeding |
| `NOTE` | Minor style observation, low urgency |

---

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill definition — trigger logic, 7-axis review, output format |
| `CHECKLIST.md` | Extended per-axis, per-language checklist for deep review passes |
| `OUTPUT_EXAMPLES.md` | 3 realistic examples: TypeScript service, Python pipeline, Go handler |
| `README.md` | This file |

---

## Scope and Non-Goals

**This skill covers only:**
- Code Claude just generated in the current response
- The 7 axes defined above

**This skill does NOT:**
- Perform full security audits
- Review architectural or business logic decisions
- Replace a proper PR review process
- Review pre-existing code that was not modified

Its only job: immediately after generating code, catch redundancy, inconsistency, compatibility issues, and waste — before you run it.

---

## Design Principles

1. **Automatic** — runs after every meaningful code generation without being asked
2. **Self-aware** — tuned for patterns AI-generated code tends to get wrong
3. **Project-aware** — considers existing conventions, not just generic best practices
4. **Concise** — scannable findings table, not a full audit report
5. **Actionable** — every finding includes a concrete fix
6. **Evidence-based** — only flags what is actually present in the generated code
