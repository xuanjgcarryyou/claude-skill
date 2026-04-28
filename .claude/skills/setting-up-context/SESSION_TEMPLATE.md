# Session Template

Use this template when manually writing a Session Brief or when CLAUDE.md / PRD.md are incomplete.

---

## Session Brief

**Stack:**
- Language: [e.g., TypeScript, Python 3.11]
- Framework: [e.g., Next.js 14, FastAPI]
- Database: [e.g., PostgreSQL 15 via Prisma]
- Test runner: [e.g., Jest, pytest]

**Build / Test Commands:**
```bash
# Dev server
[command]

# Run tests
[command]

# Build
[command]

# Lint
[command]
```

**Current Task:**
[One sentence describing what we are building or fixing today. If unknown, write UNCONFIRMED.]

**Feature Scope:**
- In scope: [list what Claude may touch]
- Out of scope: [list what Claude must not touch without asking]

**Off-Limits Zones:**
- [File or module path]: [reason]
- [File or module path]: [reason]

**Active Constraints:**
- Naming rules: [e.g., all API handlers must follow handleNoun pattern]
- Behavioral rules: [e.g., never add console.log in production code]
- Architecture rules: [e.g., no direct DB calls from route handlers]

---

## How to Use This Template

1. Copy this file to the project root or paste its content into the conversation.
2. Fill in every section — "UNCONFIRMED" is acceptable for Current Task when the PRD is not ready.
3. Share it with Claude at the start of a new session.
4. Claude will confirm the brief before beginning any work.

## What Each Field Does

| Field | Purpose |
|-------|---------|
| Stack | Prevents Claude from assuming the wrong framework or version |
| Build/Test Commands | Ensures Claude runs the correct commands, not guesses |
| Current Task | Focuses the session — Claude will not wander into adjacent features |
| Feature Scope | Defines the blast radius of permitted changes |
| Off-Limits Zones | Hard stop — Claude must ask before touching these paths |
| Active Constraints | Naming and style rules Claude must follow throughout |
