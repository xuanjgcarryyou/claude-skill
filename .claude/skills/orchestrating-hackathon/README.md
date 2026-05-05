# orchestrating-hackathon

A control-plane orchestrator for two-day team hackathons. Coordinates problem framing, architecture
design, and task decomposition so a 3-person team can go from raw prompt to parallel implementation
with minimal ambiguity and minimal merge conflicts.

---

## What It Does

This skill does NOT write business code. It manages the planning layer:

1. **Problem Framing** — transforms a vague hackathon theme into a concrete, judgeable product direction
2. **Architecture Design** — locks module boundaries, interface contracts, and ownership maps
3. **Task Generation** — produces one detailed execution package per teammate (in a fresh session)
4. **Interrupt Handling** — evaluates new ideas at any time with feasibility and impact analysis

---

## When to Use It

Trigger this skill when:
- Your team receives a hackathon prompt and needs to plan
- You want to split work cleanly to avoid merge conflicts
- You need to lock interface contracts before parallel development starts
- A teammate proposes a new feature mid-hackathon and you want to evaluate the impact

---

## How to Trigger It

Say things like:
- 「黑客松開始」
- "hackathon start"
- 「題目是…，幫我破題」
- 「我們要怎麼分工」
- "help us plan a two-day hackathon"
- "we got the theme, let's figure out what to build"

---

## Session Strategy

This skill uses **two sessions** by design:

```
Session A (Stage 1 + Stage 2)
  → Discuss problem, confirm direction, design architecture
  → Outputs: HACKATHON_BRIEF.md, ARCHITECTURE.md, TECH_STACK.md
  → End with /clear

Session B (Stage 3)
  → Fresh context: setting-up-context reads the three artifacts
  → Outputs: TASK_A.md, TASK_B.md, TASK_C.md
```

The session break prevents exploratory discussion from polluting task decomposition.

---

## What You Get

By the end of both sessions, your team has:

- **HACKATHON_BRIEF.md** — problem definition, product direction, tech choices, rationale
- **ARCHITECTURE.md** — module boundaries, data flow, interface contracts, ownership map
- **TECH_STACK.md** — confirmed stack with rationale and constraints
- **TASK_A/B/C.md** — one execution package per teammate with file-level implementation plans

Each task package is detailed enough for a coding model to implement without asking follow-up questions.

---

## Stage 4: Interrupt Handling

At any time, if a teammate proposes a new idea, this skill evaluates it against:

1. Technical feasibility (can it be built in time?)
2. Architecture impact (does it break existing boundaries?)
3. Task impact (does it invalidate someone else's package?)
4. Scoring impact (which judging dimension improves?)
5. Cost of adoption (what gets delayed or made riskier?)

Every interrupt ends with: ✅ ADOPT / ⚠️ DEFER / ❌ REJECT — never ambiguous.

---

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions — stages, governance, output rules |
| `BRIEF_TEMPLATE.md` | Output structure for `HACKATHON_BRIEF.md` (Stage 1) |
| `ARCHITECTURE_TEMPLATE.md` | Output structure for `ARCHITECTURE.md` (Stage 2) |
| `TECH_STACK_TEMPLATE.md` | Output structure for `TECH_STACK.md` (Stage 2) |
| `TASK_TEMPLATE.md` | Output structure for each `TASK_*.md` (Stage 3) |
| `README.md` | This file |

---

## Supporting Skills Used

This skill actively delegates to other skills when available:

| Skill | Used When |
|-------|----------|
| `idea-refiner` | Stage 1 problem framing |
| `auditing-dependencies` | Tech stack selection |
| `contracting-api` | Stage 2 interface design |
| `implementation-planner` | Stage 3 task generation |
| `setting-up-context` | Starting Stage 3 with clean context |
