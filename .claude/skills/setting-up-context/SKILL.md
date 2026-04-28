---
name: setting-up-context
description: >
  Session initializer that reads project configuration files and outputs a confirmed
  Session Brief before any work begins. Prevents context contamination between features
  by grounding every session in the current project state. TRIGGER at session start,
  after /clear, before switching features, or when the user says "let's start a new
  session", "initialize context", "set up context", "read the project files", "what
  are we working on", "what's our stack", "remind yourself of the project", "before
  we start", "new task context", or "fresh start". ALSO trigger automatically when
  Claude detects it is beginning work in a new conversation without having confirmed
  project context. Do NOT trigger during ongoing work within a session that already
  has a confirmed Brief.
---

# Setting Up Context

A mandatory session initializer that reads project configuration files, confirms the
current task scope, and produces a Session Brief that must be acknowledged before any
work begins.

You are NOT starting a task here. You are grounding yourself before one. Your job is
to read what the project says about itself — not infer, not assume — and reflect it
back to the user for confirmation.

---

## Purpose

Every new session risks carrying assumptions from a previous one. This skill eliminates
that risk by forcing an explicit read of the project's ground truth files before work
starts. No file reading is skipped. No assumptions fill the gaps.

---

## Execution Steps

Execute these steps in order. Do not skip any step. Do not begin work until Step 4 is confirmed.

### Step 1: Read CLAUDE.md

Locate and read `CLAUDE.md` in the project root.

Extract:
- Tech stack (languages, frameworks, runtimes, key libraries)
- Standard commands (build, test, lint, run)
- Off-limits zones (files, directories, or patterns that must not be modified)
- Naming and style conventions
- Any explicit behavioral constraints on Claude

**If CLAUDE.md does not exist:** Stop immediately. Output the following and wait:

```
CONTEXT SETUP BLOCKED

CLAUDE.md was not found in this project.

CLAUDE.md is required before work can begin. It defines the project stack,
commands, and constraints that prevent incorrect assumptions.

Please create CLAUDE.md or point to the correct project root. Do not proceed
without it.
```

Do not infer the stack from file extensions or package.json. Do not assume a stack.
Do not continue to Step 2 until CLAUDE.md exists and is read.

### Step 2: Read PRD.md

Locate and read `PRD.md` (or `PRODUCT_REQUIREMENTS.md` if PRD.md is absent).

Extract:
- Current active goal or feature
- Feature scope boundaries (what is in scope, what is explicitly out of scope)
- Any stated MVP constraints

**If PRD.md does not exist:** Note it as absent. Continue to Step 3. At Session Brief
output time, mark "Current Task" as UNCONFIRMED.

### Step 3: Read TechDesign.md (conditional)

Check for `TechDesign.md` (or `TECH_DESIGN.md`, `ARCHITECTURE.md`) in the project root.

**If it exists:** Read and extract key architectural decisions and component constraints.

**If it does not exist:** Skip silently.

### Step 4: Output Session Brief and Wait

Produce the Session Brief (format below). Output it and stop.

Do not begin any work. Do not propose solutions. Do not write code.
Wait for the user to explicitly confirm or correct the Brief.

Only after the user confirms — by saying "confirmed", "looks good", "yes", "correct",
or providing corrections — may work begin.

---

## Session Brief Format

Output exactly this structure. Do not add sections. Do not omit sections.

```
## Session Brief

**Stack:** [languages, frameworks, runtime versions from CLAUDE.md]
**Build/Test Commands:** [commands from CLAUDE.md]
**Current Task:** [goal from PRD.md — or UNCONFIRMED if PRD.md absent]
**Feature Scope:** [what is in scope / out of scope for current task]
**Off-limits Zones:** [files, directories, or patterns listed in CLAUDE.md]
**Active Constraints:** [naming conventions, behavioral rules, architectural decisions]

---

Ready to proceed. Please confirm the above is correct, or tell me what to correct.
```

If a field has no data, write "None declared" — do not omit the field.

If Current Task is UNCONFIRMED, write:
`UNCONFIRMED — please state the task before confirming this Brief`

---

## Confirmation Protocol

- If the user confirms: acknowledge and ask for the first task.
- If the user corrects something: update the Brief, output the corrected version, and wait again.
- If the user asks a clarifying question: answer it, but do not begin work until the Brief is confirmed.

---

## What This Skill Does NOT Do

- Does not infer project stack from file contents or project structure
- Does not begin planning, designing, or writing code before confirmation
- Does not skip CLAUDE.md — there is no fallback to assumption
- Does not carry over task context from a previous session

---

## Extended Resources

- `SESSION_TEMPLATE.md` — blank Session Brief template for manual completion
- `README.md` — purpose, workflow position, and configuration guidance
