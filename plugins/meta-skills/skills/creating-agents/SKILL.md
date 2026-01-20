---
name: creating-agents
description: Use this skill when creating custom Claude Code subagents, designing agent configurations, deciding between agent scopes (user vs project), configuring tool restrictions, or setting up lifecycle hooks. Helps design focused agents with appropriate capabilities.
---

# Creating Custom Claude Code Subagents

This skill guides you through designing and creating custom subagent configurations for Claude Code CLI.

## Official Documentation

For the most current information, reference:
- https://docs.anthropic.com/en/docs/claude-code/sub-agents

## When to Create a Custom Agent

**Use Built-in Agents When:**
- **Explore**: Fast, read-only codebase exploration (uses Haiku)
- **Plan**: Research and planning tasks
- **General-purpose**: Complex multi-step tasks with full capabilities
- **Bash**: Terminal commands in separate context

**Create Custom Agents When:**
- You need a specialized domain expert (code reviewer, data scientist, security auditor)
- You require specific tool restrictions for safety or focus
- You want custom hooks for validation or lifecycle events
- You need a focused system prompt for consistent behavior
- You want to share configurations with your team via version control

## Agent Design Process

### 1. Gather Requirements

Ask the user to clarify:
- What specific task should this agent handle?
- What tools does it need? What tools should it NOT have?
- Should this be user-level (all projects) or project-level?
- Does it need hooks for validation or setup/cleanup?

### 2. Choose Location

| Location | Path | Scope | Use Case |
|----------|------|-------|----------|
| Project | `.claude/agents/` | Current project | Team-shared, version-controlled |
| User | `~/.claude/agents/` | All your projects | Personal preferences |
| Session | `--agents` CLI flag | Current session only | Temporary experimentation |

### 3. Write the Description (Critical!)

The description determines when Claude delegates to your agent. Write it like you're telling Claude: "Use this agent when..."

**Good descriptions:**
- "Expert code review specialist. Proactively reviews code for quality, security, and maintainability"
- "Execute read-only database queries with SQL validation"
- "Debugging specialist for errors, test failures, and unexpected behavior"

**Bad descriptions:**
- "Code helper" (too vague)
- "Does stuff with files" (no trigger keywords)
- "My agent" (meaningless)

### 4. Configure Tool Access

Three approaches:
- **Inherit all** (default): Omit `tools` field entirely
- **Allowlist**: Use `tools` to specify only permitted tools
- **Denylist**: Use `disallowedTools` to remove specific tools

Common tool sets:
- **Read-only**: `Read, Grep, Glob, Bash`
- **With editing**: `Read, Edit, Write, Bash, Grep, Glob`
- **Minimal**: `Read, Grep, Glob` (no Bash)

### 5. Set Permission Mode

| Mode | Behavior |
|------|----------|
| `default` | Standard permission prompts |
| `acceptEdits` | Auto-accept file edits |
| `dontAsk` | Auto-deny permission prompts |
| `bypassPermissions` | Skip all checks (use cautiously) |
| `plan` | Read-only exploration mode |

### 6. Add Hooks (Optional)

Hooks enable validation or lifecycle events. See `./references/hooks-and-validation-patterns.md` for examples.

## Quick Configuration Reference

```yaml
---
name: my-agent                    # Required: lowercase, hyphens only
description: When to use this     # Required: determines delegation
model: sonnet                     # Optional: sonnet|opus|haiku|inherit
tools: Read, Grep, Glob           # Optional: allowlist (inherits all if omitted)
disallowedTools: Write, Edit      # Optional: denylist
permissionMode: default           # Optional: permission handling
skills: my-skill, other-skill     # Optional: preload skill context
hooks:                            # Optional: lifecycle hooks
  PreToolUse: [...]
  PostToolUse: [...]
---

Your system prompt goes here. This is the context and instructions
the agent receives when invoked.
```

For detailed field documentation, see `./references/agent-configuration-options.md`.

## Common Patterns

### Read-Only Reviewer
```yaml
---
name: code-reviewer
description: Expert code review specialist for quality, security, and maintainability
tools: Read, Grep, Glob, Bash
model: inherit
---

You are a senior code reviewer. Analyze code and provide specific, actionable feedback.
```

### Database Reader with Validation
```yaml
---
name: db-reader
description: Execute read-only database queries with SQL validation
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You execute SQL queries. Only SELECT statements are permitted.
```

### Focused Domain Specialist
```yaml
---
name: security-auditor
description: Security specialist for vulnerability assessment and secure coding review
tools: Read, Grep, Glob, Bash
disallowedTools: Edit, Write
---

You are a security expert. Focus on OWASP Top 10, authentication, authorization,
input validation, and secure data handling.
```

For more examples, see `./references/agent-examples.md`.

## Validation Checklist

Before finalizing your agent:

- [ ] **Name**: Lowercase letters and hyphens only, unique identifier
- [ ] **Description**: Clear, specific, includes trigger keywords
- [ ] **Tools**: Minimally scoped for the task (principle of least privilege)
- [ ] **System prompt**: Focused instructions for the agent's role
- [ ] **Location**: Appropriate scope (project vs user)
- [ ] **Hooks**: If used, validation script tested and working

## Creating the Agent File

1. Create the file at the appropriate location:
   ```bash
   # Project-level (version-controlled)
   mkdir -p .claude/agents

   # User-level (personal)
   mkdir -p ~/.claude/agents
   ```

2. Write the agent file with `.md` extension:
   ```bash
   # Example: .claude/agents/code-reviewer.md
   ```

3. Test by asking Claude to use the agent or let it auto-delegate based on description

## Using Agents

Once created, agents are available automatically:
- Claude delegates based on matching descriptions
- Explicitly invoke: "Use the code-reviewer agent to analyze this file"
- Manage via `/agents` command in Claude Code

## Reference Files

- `./references/agent-configuration-options.md` — All frontmatter fields explained
- `./references/hooks-and-validation-patterns.md` — Hook examples and validation scripts
- `./references/agent-examples.md` — Curated agent configurations for common use cases
