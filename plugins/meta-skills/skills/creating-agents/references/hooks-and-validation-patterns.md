# Hooks and Validation Patterns

Hooks allow agents to run shell commands at specific lifecycle points. Use them for validation, logging, setup, and cleanup.

## Hook Types

### PreToolUse

Runs before a tool executes. Can block the operation.

**Exit Codes:**
- `0` - Allow the operation
- `2` - Block the operation (with stderr as reason)
- Other - Allow but log warning

**Input:** JSON object with tool details piped to stdin

```json
{
  "tool_name": "Bash",
  "tool_input": {
    "command": "rm -rf /tmp/test"
  }
}
```

### PostToolUse

Runs after a tool completes. Cannot block, but can perform side effects.

**Input:** JSON object with tool details and result

```json
{
  "tool_name": "Edit",
  "tool_input": { "file_path": "/src/main.js", "..." },
  "tool_result": { "success": true }
}
```

### Stop

Runs when the agent completes. Use for cleanup.

**Input:** JSON object with completion details

## Validation Patterns

### Read-Only Database Queries

Block write operations while allowing SELECT queries.

**Agent configuration:**
```yaml
---
name: db-reader
description: Execute read-only database queries with validation
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
---

You execute SQL queries against the database. Only SELECT statements are permitted.
All write operations (INSERT, UPDATE, DELETE, DROP, etc.) will be blocked.
```

**Validation script (`./scripts/validate-readonly-query.sh`):**
```bash
#!/bin/bash
set -e

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Block SQL write operations
if echo "$COMMAND" | grep -iE '\b(INSERT|UPDATE|DELETE|DROP|CREATE|ALTER|TRUNCATE|GRANT|REVOKE)\b' > /dev/null; then
  echo "Blocked: Only SELECT queries are allowed. Detected write operation." >&2
  exit 2
fi

exit 0
```

### Command Allowlist

Only permit specific commands.

**Agent configuration:**
```yaml
---
name: safe-executor
description: Execute pre-approved commands only
tools: Bash
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-allowed-commands.sh"
---
```

**Validation script:**
```bash
#!/bin/bash
set -e

INPUT=$(cat)
COMMAND=$(echo "$INPUT" | jq -r '.tool_input.command // empty')

# Allowed command prefixes
ALLOWED=(
  "npm test"
  "npm run lint"
  "git status"
  "git diff"
  "git log"
)

for prefix in "${ALLOWED[@]}"; do
  if [[ "$COMMAND" == "$prefix"* ]]; then
    exit 0
  fi
done

echo "Blocked: Command not in allowlist. Permitted: ${ALLOWED[*]}" >&2
exit 2
```

### File Path Restrictions

Limit file access to specific directories.

**Agent configuration:**
```yaml
---
name: src-only-editor
description: Edit files only within the src directory
tools: Read, Edit, Write, Grep, Glob
hooks:
  PreToolUse:
    - matcher: "Edit|Write"
      hooks:
        - type: command
          command: "./scripts/validate-file-path.sh"
---
```

**Validation script:**
```bash
#!/bin/bash
set -e

INPUT=$(cat)
FILE_PATH=$(echo "$INPUT" | jq -r '.tool_input.file_path // empty')

# Only allow files in src/
if [[ "$FILE_PATH" != */src/* ]] && [[ "$FILE_PATH" != src/* ]]; then
  echo "Blocked: Can only edit files within src/ directory" >&2
  exit 2
fi

exit 0
```

## Logging and Auditing

### Log All Tool Usage

**Agent configuration:**
```yaml
---
name: audited-agent
description: Agent with full audit logging
hooks:
  PostToolUse:
    - matcher: ".*"
      hooks:
        - type: command
          command: "./scripts/log-tool-usage.sh"
---
```

**Logging script:**
```bash
#!/bin/bash
INPUT=$(cat)
TIMESTAMP=$(date -u +"%Y-%m-%dT%H:%M:%SZ")
TOOL_NAME=$(echo "$INPUT" | jq -r '.tool_name')

echo "[$TIMESTAMP] Tool used: $TOOL_NAME" >> ~/.claude/audit.log
echo "$INPUT" | jq -c '.' >> ~/.claude/audit-details.log
```

## Setup and Cleanup

### Database Connection Setup

**Agent configuration:**
```yaml
---
name: db-analyst
description: Database analyst with connection setup
tools: Bash, Read, Write
hooks:
  Stop:
    - hooks:
        - type: command
          command: "./scripts/cleanup-db-connection.sh"
---
```

**Project-level hooks (`.claude/settings.json`):**
```json
{
  "hooks": {
    "SubagentStart": [
      {
        "matcher": "db-analyst",
        "hooks": [
          { "type": "command", "command": "./scripts/setup-db-connection.sh" }
        ]
      }
    ],
    "SubagentStop": [
      {
        "matcher": "db-analyst",
        "hooks": [
          { "type": "command", "command": "./scripts/cleanup-db-connection.sh" }
        ]
      }
    ]
  }
}
```

## Combining Multiple Validations

**Agent configuration:**
```yaml
---
name: safe-data-agent
description: Data agent with multiple safety checks
tools: Bash, Read
hooks:
  PreToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/validate-readonly-query.sh"
        - type: command
          command: "./scripts/validate-allowed-tables.sh"
  PostToolUse:
    - matcher: "Bash"
      hooks:
        - type: command
          command: "./scripts/log-query.sh"
---
```

## Testing Hooks

Before deploying an agent with hooks:

1. **Test validation scripts standalone:**
   ```bash
   echo '{"tool_name":"Bash","tool_input":{"command":"SELECT * FROM users"}}' | ./scripts/validate-readonly-query.sh
   echo $?  # Should be 0

   echo '{"tool_name":"Bash","tool_input":{"command":"DELETE FROM users"}}' | ./scripts/validate-readonly-query.sh
   echo $?  # Should be 2
   ```

2. **Check stderr output for blocked operations:**
   ```bash
   echo '{"tool_name":"Bash","tool_input":{"command":"DROP TABLE users"}}' | ./scripts/validate-readonly-query.sh 2>&1
   # Should show: "Blocked: Only SELECT queries are allowed..."
   ```

3. **Test with the agent:** Create a test scenario and verify hooks trigger correctly.

## Hook Best Practices

1. **Keep hooks fast** - They run synchronously and block tool execution
2. **Handle errors gracefully** - Use `set -e` and provide clear error messages
3. **Use jq for JSON parsing** - Reliable and consistent
4. **Log important events** - Audit trails help debugging
5. **Test thoroughly** - Validate both allow and block cases
6. **Scope narrowly** - Use specific matchers rather than `.*`
