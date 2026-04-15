# Skills Registry

A tracking file for all custom Claude Code skills in this project.

## Skills

| Skill | Trigger | Status | Location |
|-------|---------|--------|----------|
| auto-code-review | Automatically after every meaningful code generation/modification | Active | `skills/auto-code-review/` |
| idea-refiner | "I have an idea", "help me think through this", "how should I architect...", "let's plan this out" | Active | `skills/idea-refiner/` |

## Notes

- All skills live under `.claude/skills/`
- Each skill folder = one skill, must contain a `SKILL.md` with YAML frontmatter (`name`, `description`)
- See `../anthropics-skills/SKILL.md` for the skill creator reference
