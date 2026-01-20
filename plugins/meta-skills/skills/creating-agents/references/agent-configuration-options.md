# Agent Configuration Options

Complete reference for all subagent frontmatter fields.

## Required Fields

### `name`

Unique identifier for the agent.

**Requirements:**
- Lowercase letters and hyphens only
- Must be unique within scope
- Used for invocation and management

**Examples:**
```yaml
name: code-reviewer
name: db-reader
name: security-auditor
```

### `description`

Determines when Claude delegates to this agent. This is the most critical field.

**Best Practices:**
- Write in third person
- Include trigger keywords
- Be specific about use cases
- Think: "When would Claude know to use this?"

**Examples:**
```yaml
# Good - specific, includes triggers
description: Expert code review specialist. Proactively reviews code for quality, security, and maintainability

# Good - clear use case
description: Execute read-only database queries with SQL validation

# Bad - too vague
description: Helps with code

# Bad - no trigger keywords
description: My custom agent
```

## Optional Fields

### `model`

Which Claude model the agent uses.

| Value | Description |
|-------|-------------|
| `sonnet` | Claude Sonnet (default) |
| `opus` | Claude Opus (most capable) |
| `haiku` | Claude Haiku (fastest, cheapest) |
| `inherit` | Same as parent conversation |

**Usage:**
```yaml
model: haiku    # For fast, simple tasks
model: opus     # For complex reasoning
model: inherit  # Match parent context
```

**Guidelines:**
- Use `haiku` for high-volume, simple tasks (exploration, formatting)
- Use `sonnet` for most tasks (good balance)
- Use `opus` for complex reasoning or critical decisions
- Use `inherit` when consistency with parent matters

### `tools`

Allowlist of tools the agent can use. If omitted, inherits all tools from parent.

**Available Tools:**
- `Read` - Read files
- `Write` - Create/overwrite files
- `Edit` - Modify existing files
- `Glob` - Find files by pattern
- `Grep` - Search file contents
- `Bash` - Execute shell commands
- `WebFetch` - Fetch web content
- `WebSearch` - Search the web
- `Task` - Launch subagents
- `TodoWrite` - Manage task lists
- `NotebookEdit` - Edit Jupyter notebooks

**Usage:**
```yaml
# Read-only agent
tools: Read, Grep, Glob

# Read-only with shell access
tools: Read, Grep, Glob, Bash

# Full file access
tools: Read, Write, Edit, Grep, Glob, Bash
```

### `disallowedTools`

Denylist of tools to remove from inherited set.

**Usage:**
```yaml
# Inherit all except file writing
disallowedTools: Write, Edit

# No web access
disallowedTools: WebFetch, WebSearch

# No subagent spawning
disallowedTools: Task
```

**Note:** Use `tools` (allowlist) OR `disallowedTools` (denylist), not both.

### `permissionMode`

Controls how the agent handles permission prompts.

| Mode | Behavior | Use Case |
|------|----------|----------|
| `default` | Standard permission prompts | Most agents |
| `acceptEdits` | Auto-accept file edits | Trusted editing agents |
| `dontAsk` | Auto-deny permission prompts | Strict read-only |
| `bypassPermissions` | Skip all permission checks | Fully automated (use cautiously) |
| `plan` | Read-only exploration mode | Research agents |

**Usage:**
```yaml
# Standard (default)
permissionMode: default

# Auto-accept edits for trusted refactoring agent
permissionMode: acceptEdits

# Read-only research mode
permissionMode: plan
```

**Security Warning:** `bypassPermissions` should only be used for fully trusted, well-tested agents in controlled environments.

### `skills`

Skills to preload into the agent's context at startup.

**Usage:**
```yaml
# Single skill
skills: processing-pdfs

# Multiple skills
skills: processing-pdfs, analyzing-data
```

**Note:** Skills must be installed in `~/.claude/skills/` or `.claude/skills/`.

### `hooks`

Lifecycle hooks for validation, setup, or cleanup. Hooks run shell commands at specific points.

**Hook Types:**

| Hook | Trigger | Use Case |
|------|---------|----------|
| `PreToolUse` | Before a tool executes | Validation, blocking |
| `PostToolUse` | After a tool executes | Logging, side effects |
| `Stop` | When agent completes | Cleanup |

**Usage:**
```yaml
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-command.sh"
  PostToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/run-linter.sh"
```

See `./hooks-and-validation-patterns.md` for detailed examples.

## Complete Example

```yaml
---
name: data-analyst
description: Data analysis expert for SQL queries, data exploration, and statistical insights
model: sonnet
tools: Bash, Read, Write, Grep, Glob
permissionMode: default
skills: analyzing-data
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-query.sh"
---

You are a data analyst specializing in SQL and data exploration.

Your responsibilities:
- Write optimized SQL queries
- Analyze query results and identify patterns
- Provide data-driven recommendations
- Create clear visualizations when helpful

Always explain your methodology and any assumptions made.
```

## Field Summary Table

| Field | Required | Type | Default |
|-------|----------|------|---------|
| `name` | Yes | string | - |
| `description` | Yes | string | - |
| `model` | No | enum | `sonnet` |
| `tools` | No | list | inherit all |
| `disallowedTools` | No | list | none |
| `permissionMode` | No | enum | `default` |
| `skills` | No | list | none |
| `hooks` | No | object | none |
