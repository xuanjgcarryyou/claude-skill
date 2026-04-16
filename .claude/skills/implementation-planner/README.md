# Implementation Planner

A Claude Code skill that turns architecture specs into executable implementation plans — researching the best approach for each component, mapping dependencies, and producing task packages that subagents or Claude Code sessions can execute independently.

---

## What It Does

This skill takes a completed architecture spec or system design and produces:

1. **Component analysis** — the best implementation method for each component, with rationale
2. **Dependency map** — what needs what, what can run in parallel
3. **Phased build order** — foundation → core → integration → secondary
4. **Task packages** — self-contained work tickets, one per component or major operation

The task packages are the main deliverable. Each one is written so a Claude Code session or subagent can pick it up, open the referenced files, and start executing without needing to read this planning session or ask follow-up questions.

---

## Where It Sits in the Workflow

```
[Raw Idea]
    ↓
idea-refiner        ← clarifies, proposes architecture, produces spec
    ↓
implementation-planner  ← THIS SKILL: researches, plans, decomposes into tasks
    ↓
[Code generation sessions / subagents]  ← execute task packages
```

This skill sits between architecture design and code generation. It does not generate code and does not brainstorm ideas. Its only concern is: given a design, how do we actually build each piece, in what order, and what does each builder need?

---

## When to Use It

Use this skill when:
- You have a completed architecture spec and want to know how to actually build each part
- You want to split a system into tasks for separate Claude Code sessions
- You want Claude to research the best method/library/approach for each component before writing code
- You want to know which components can be built in parallel vs. sequentially
- You want clear task scopes so each coding session has exactly what it needs — no more, no less

**Do not use this skill for:**
- Raw, undefined ideas (use `idea-refiner` first)
- Writing or reviewing code
- Project timelines or estimates

---

## How to Use It

Simply paste or describe your architecture spec and say something like:
- "how do we build this"
- "break this into tasks"
- "give me a task breakdown"
- "split this for subagents"
- "what's the build plan"

The skill will produce a full implementation plan immediately — no back-and-forth needed.

If your spec is incomplete or ambiguous in places, the skill will flag those as Pre-Build Decisions rather than inventing requirements.

---

## Output Structure

```
Implementation Plan
├── Architecture Summary       — recap of what was designed
├── Stack Confirmation         — confirmed tech stack
├── Component Analysis         — chosen approach + rationale per component
├── Dependency Map             — what depends on what
├── Build Phases               — phase 1–4 grouping
├── Parallelization            — which tasks can run simultaneously
├── Task Packages              — one full task package per task
├── Integration Checklist      — seams between components to watch
└── Pre-Build Decisions        — decisions that must be made before coding starts
```

---

## Task Package Structure

Each task package includes:
- Phase and parallel/sequential relationship to other tasks
- Objective (one sentence)
- Scope (included and explicitly excluded)
- Input (what must exist before this task starts)
- Output (what this task produces)
- Implementation method and key library/pattern
- Interface contract (function signatures, endpoint shapes, event names)
- Constraints
- Context for the subagent (relevant spec excerpts, schema snippets)
- Risks and unknowns

See `TASK_TEMPLATE.md` for the full template and field-by-field instructions.

---

## Files in This Skill

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill definition — behavior, output format, rules |
| `TASK_TEMPLATE.md` | Reusable template for a single task package with instructions |
| `OUTPUT_EXAMPLES.md` | 2 full worked examples (simple 4-task system, complex 6-task system) |
| `README.md` | This file |
