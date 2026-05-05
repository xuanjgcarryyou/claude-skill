# Skills Registry

A tracking file for all custom Claude Code skills in this project.

## Skills

| Skill | Trigger | Status | Location |
|-------|---------|--------|----------|
| auto-code-review | Automatically after every meaningful code generation/modification | Active | `skills/auto-code-review/` |
| auditing-dependencies | "install", "npm install", "add dependency", "import", "new package", "pip install" | Active | `skills/auditing-dependencies/` |
| contracting-api | "define API", "lock the interface", "API contract", "frontend-backend boundary", "agree on the shape" | Active | `skills/contracting-api/` |
| debugging-systematically | "still broken", "tried three times", "I don't know why", "same error", "still failing" | Active | `skills/debugging-systematically/` |
| idea-refiner | "I have an idea", "help me think through this", "how should I architect...", "let's plan this out" | Active | `skills/idea-refiner/` |
| implementation-planner | "how do we build this", "break this into tasks", "give me a task breakdown", "split this for subagents", "what's the build plan" | Active | `skills/implementation-planner/` |
| migrating-schema | "migration", "alter table", "schema change", "add column", "drop column", "schema.prisma" | Active | `skills/migrating-schema/` |
| orchestrating-hackathon | "黑客松開始", "hackathon start", "題目是…", "幫我破題", "我們要參加黑客松", "two-day hackathon" | Active | `skills/orchestrating-hackathon/` |
| refactoring-safely | "refactor", "clean up", "restructure", "reorganize", "simplify the code" | Active | `skills/refactoring-safely/` |
| reviewing-security | "security review", "security audit", "is this secure", "review for security", "before we ship" | Active | `skills/reviewing-security/` |
| running-retrospective | "retro", "retrospective", "let's do a retro", "what did we learn", "update CLAUDE.md" | Active | `skills/running-retrospective/` |
| setting-up-context | "let's start a new session", "initialize context", "set up context", "what are we working on", "fresh start" | Active | `skills/setting-up-context/` |
| summarizing-pr | "write PR description", "summarize my changes", "create PR", "PR ready", "write the PR" | Active | `skills/summarizing-pr/` |
| writing-tests | "write tests", "add tests", "test coverage", "generate tests", "missing tests" | Active | `skills/writing-tests/` |

## Notes

- All skills live under `.claude/skills/`
- Each skill folder = one skill, must contain a `SKILL.md` with YAML frontmatter (`name`, `description`)
- See `../anthropics-skills/SKILL.md` for the skill creator reference
