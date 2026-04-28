# setting-up-context

Session initializer — grounds every session in the current project state before any work begins.

## When to Use

Say any of the following to activate:
- "let's start a new session"
- "set up context"
- "what's our stack"
- "before we start"
- "initialize context"
- "remind yourself of the project"

Also activates automatically when Claude detects it is starting work without a confirmed project context.

## What It Does

1. Reads `CLAUDE.md` (required), `PRD.md` (optional), and `TechDesign.md` (optional)
2. Outputs a **Session Brief** with six fixed fields
3. Waits for your confirmation before any work begins

## Session Brief Format

```
## Session Brief
Stack:              [framework, language, DB]
Build/Test:         [commands]
Current Task:       [from PRD.md, or UNCONFIRMED]
Feature Scope:      [in / out of scope]
Off-limits Zones:   [from CLAUDE.md]
Active Constraints: [naming rules, behavioral rules]

Ready to proceed. Please confirm or correct any of the above.
```

## Failure Modes

| Situation | Behaviour |
|-----------|-----------|
| `CLAUDE.md` missing | Hard stop — outputs `CONTEXT SETUP BLOCKED`, asks you to create it |
| `PRD.md` missing | Continues with `UNCONFIRMED` in Current Task field |
| Both present but incomplete | Outputs Brief with `[unknown]` placeholders, asks you to correct |

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill instructions loaded by Claude Code |
| `SESSION_TEMPLATE.md` | Blank template to fill in manually when CLAUDE.md is incomplete |
