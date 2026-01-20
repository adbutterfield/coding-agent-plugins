# Coding Agent Plugins

A marketplace of skill plugins for coding agents using Claude Code CLI.

## Available Plugins

### coding-skills

Practical skills for coding agents: React/Next.js optimization and safe Vitest execution.

| Skill                     | Description                                                                                                                                                                |
| ------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **developing-react-apps** | 47 performance optimization rules for React and Next.js applications covering async patterns, bundle optimization, server/client rendering, re-render prevention, and more |
| **running-vitest-safely** | Safe Vitest test execution with proper flags to prevent hung processes, manage watch mode, set timeouts, and ensure proper cleanup                                         |

### meta-skills

Skills for working with Claude Code: creating skills, creating custom agents, and orchestrating task agents.

| Skill                         | Description                                                                                                                                                             |
| ----------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **creating-agents**           | Create custom Claude Code subagents with appropriate scopes, tool restrictions, and lifecycle hooks for specialized domain experts                                      |
| **creating-skills**           | Create new Claude Code skills, edit existing skills, or convert sub-agents to skills. Includes guidance on SKILL.md structure, naming conventions, and best practices   |
| **orchestrating-task-agents** | Best practices for delegating work to Task agents including XML prompt structure, agent type selection, parallel/sequential coordination, and skill invocation patterns |

## Installation

Add the marketplace to Claude Code:

```bash
/plugin marketplace add adbutterfield/coding-agent-plugins
```

Then install the plugins you want:

```bash
/plugin install coding-skills@coding-agent-plugins
/plugin install meta-skills@coding-agent-plugins
```

Or install both:

```bash
/plugin install coding-skills@coding-agent-plugins meta-skills@coding-agent-plugins
```

## Requirements

- Claude Code v1.0.33 or later

## Acknowledgments

The `developing-react-apps` skill was initially based on [react-best-practices](https://github.com/vercel-labs/agent-skills/tree/main/skills/react-best-practices) from Vercel Labs. It has since been revised to follow official [Claude Code skill authoring best practices](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices) and the [Agent Skills Specification](https://agentskills.io/specification), along with personal experience crafting skills that agents can effectively use.

## License

MIT
