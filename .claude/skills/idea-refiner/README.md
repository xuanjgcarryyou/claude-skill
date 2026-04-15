# idea-refiner

A technical thinking partner that turns raw ideas into executable build plans.

---

## What It Does

When you have a rough idea, this skill helps you think it through before writing any code. It:

1. Asks ONE focused question at a time to clarify what matters
2. Proposes 2–3 architecture options with honest trade-offs
3. Helps you pick or combine approaches
4. Produces a concrete, actionable specification you can hand directly to Claude Code

---

## When to Use It

Use this skill when you:
- Have a new idea and want to think through the architecture
- Are unsure how to structure something before building
- Want to define scope (what's in v1, what's out)
- Want to see trade-offs between different approaches
- Need a spec to guide your coding session

---

## When NOT to Use It

This skill is not for:
- Writing code (it doesn't generate code)
- Reviewing existing code
- Debugging
- Project management or timelines

---

## How to Trigger It

Say things like:
- "I have an idea..."
- "I want to build something that..."
- "Help me think through this"
- "How should I architect..."
- "Let's plan this before we code"
- "I'm thinking about building..."
- "Help me scope this"

---

## What You Get

After a short conversation (2–4 exchanges), you get:
- A structured **Idea Specification** with goals, components, data flow, API outline, tech stack recommendation, risks, and build order
- Clear architecture options with trade-offs before you commit
- An MVP scope you can actually ship

---

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Main skill instructions — core behavior, interaction rules, output format |
| `QUESTION_BANK.md` | Reference bank of clarifying questions organized by category |
| `OUTPUT_EXAMPLES.md` | Two worked examples (simple CLI tool + complex Slack bot) from raw idea to final spec |
