# References Index

This directory contains detailed reference documentation for creating Claude Code skills.

## Reference Files

| File | Description |
|------|-------------|
| [skill-structure-and-format.md](skill-structure-and-format.md) | Complete SKILL.md format, directory structure, YAML frontmatter rules |
| [metadata-requirements.md](metadata-requirements.md) | Detailed guidance on `name` and `description` fields with examples |
| [skill-best-practices.md](skill-best-practices.md) | Core philosophy, structural patterns, progressive disclosure |
| [editing-skills-guide.md](editing-skills-guide.md) | How to refine existing skills, common improvements |
| [nodejs-and-cli-patterns.md](nodejs-and-cli-patterns.md) | CLI tool examples (gh, aws, npm), Node.js scripting patterns |
| [skill-template.md](skill-template.md) | Starter template for new SKILL.md files |
| [sub-agent-conversion-guide.md](sub-agent-conversion-guide.md) | Overview of converting sub-agents to skills, key differences |
| [sub-agent-conversion-examples.md](sub-agent-conversion-examples.md) | Three detailed before/after conversion examples |
| [sub-agent-conversion-checklist.md](sub-agent-conversion-checklist.md) | Conversion checklist, testing, troubleshooting |

## When to Use Each

- **Creating a new skill?** Start with `skill-template.md`, reference `skill-structure-and-format.md`
- **Skill not being invoked?** See `metadata-requirements.md` for description writing guidance
- **SKILL.md too long?** See `skill-best-practices.md` for progressive disclosure patterns
- **Improving an existing skill?** See `editing-skills-guide.md`
- **Adding scripts?** See `nodejs-and-cli-patterns.md` for CLI and Node.js examples
- **Converting a sub-agent?** Start with `sub-agent-conversion-guide.md`, see examples in `sub-agent-conversion-examples.md`
