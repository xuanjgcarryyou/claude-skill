# running-retrospective

Session Knowledge Archaeologist — extracts durable rules from successes and failures, then encodes them into CLAUDE.md.

## When to Use

Say any of the following to activate:
- "retro"
- "retrospective"
- "let's do a retro"
- "what did we learn"
- "update CLAUDE.md"
- "session wrap-up"

## What It Does

Six phases:

| Phase | Output |
|-------|--------|
| 1 — Archaeology (silent) | Reviews full conversation; collects 5 categories of data points |
| 2 — Effective Patterns | 2–8 reusable prompt patterns with reusability scores |
| 3 — Misjudgement Log | Every AI error with category, evidence, and a proposed CLAUDE.md rule |
| 4 — CLAUDE.md Diff | Ready-to-apply diff with evidence citations on every new rule |
| 5 — Off-Limits Recommendations | New zones to add, existing zones to relax or remove |
| 6 — Session Score + Primer | 2-3 sentence summary to paste into the next session's first message |

## Misjudgement Categories

| Category | Example |
|----------|---------|
| Scope-Creep | Claude added unrequested files or refactors |
| Wrong-Assumption | Claude assumed incorrect facts |
| Over-Engineering | Claude proposed unneeded abstractions |
| Hallucination | Claude referenced non-existent functions or APIs |
| Premature-Action | Claude made changes before getting confirmation |
| Missing-Confirmation | Claude should have asked before proceeding |

## Rule Quality Gate

Every proposed rule must be:
- **Imperative**: starts with NEVER or ALWAYS
- **Specific**: names a concrete action and condition
- **Evidence-backed**: cites the session interaction that motivated it

"Be more careful" → rejected. "NEVER add console.log to production code" → accepted.

## Files

| File | Purpose |
|------|---------|
| `SKILL.md` | Skill instructions |
| `RETRO_GUIDE.md` | How to apply the diff, category guide, good vs. bad rule examples |
