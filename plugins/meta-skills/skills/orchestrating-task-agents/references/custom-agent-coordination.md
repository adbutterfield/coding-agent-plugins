# Custom Agent Coordination

Patterns for coordinating user-defined agents (configured in `agents/` directory) with the main orchestrator agent.

---

## Understanding Tool Access

Custom agents have tool configurations specified in their frontmatter:

```yaml
---
name: code-review
tools: Read, Glob, Grep, Bash  # No Write/Edit
---
```

If an agent lacks a tool, it will:
- Use workarounds (e.g., `cat` heredocs instead of Write)
- Trigger permission prompts for bash operations
- Potentially fail or work inefficiently

### Common Tool Configurations

| Agent Role | Typical Tools | Has Write? |
|------------|---------------|------------|
| Code review | Read, Glob, Grep, Bash | No |
| Tech lead | Read, Glob, Grep, Bash, WebSearch | No |
| Coding/Implementation | Read, Write, Edit, Glob, Grep, Bash | Yes |
| Research/Analysis | Read, Glob, Grep, WebSearch, WebFetch | No |

---

## Coordination Pattern: Agent Returns Content, Main Writes

When a task agent lacks Write access but needs to produce files:

```
1. Main agent creates branch, does initial setup (already approved)
2. Task agent does analysis, returns file contents as structured data
3. Main agent writes files using its approved Write tool
4. Task agent verifies results via Read (optional)
```

### Task Prompt for Content Generation

```
<task>
Generate the content for [description].
Do NOT write files directly - return the content in your response.
</task>

<output_format>
Return as structured data:

<file path="path/to/file.ts">
[file content here]
</file>

Summary: [what was generated]
</output_format>
```

See `task-prompt-templates.md` for full template.

---

## Pre-Spawn Verification (Optional)

Before spawning a task agent, the main agent can check if the agent has necessary tools:

```
Before spawning [agent-name] for [task]:
1. Read .claude/agents/[agent-name].md
2. Check tools: line in frontmatter
3. If task requires Write and agent lacks it:
   - Use coordination pattern (agent returns content)
   - Or use general-purpose agent instead
```

**Limitation:** It's not always possible to anticipate all tools a task will need.

---

## Post-Task Feedback Loop

A more robust approach: have task agents report friction encountered during execution.

### Add to Agent Return Format

Include a "Friction Report" section in task agent responses:

```markdown
## Friction Report (if any)

- **Tool access:** [e.g., "Used cat heredoc for file creation - no Write access"]
- **Permission prompts:** [e.g., "3 prompts for npm commands"]
- **Workarounds used:** [e.g., "Had to split task into smaller chunks"]
- **Suggestion:** [e.g., "Add Write to tools config" or "Pre-approve npm test"]
```

### Using Feedback

When task agents report friction:

1. **Immediate:** Main agent can handle remaining operations that need approval
2. **Session:** Note patterns for current workflow adjustments
3. **Permanent:** Update agent configurations or coordination patterns

### Example Feedback

```markdown
## Friction Report

- **Tool access:** No Write tool - used `cat <<'EOF'` for 3 files, each required approval
- **Permission prompts:** 3 bash write operations
- **Suggestion:** Either add Write to code-review agent, or have main agent
  handle file creation based on my returned content
```

---

## When to Use Custom Agents vs Built-in

| Scenario | Recommendation |
|----------|----------------|
| Specialized domain knowledge (Phaser, specific framework) | Custom agent with tailored instructions |
| One-off implementation task | Built-in `general-purpose` |
| Read-only analysis/review | Custom agent without Write (by design) |
| Task requires file creation | Custom agent with Write, OR coordination pattern |

### Signs You Need the Coordination Pattern

- Task agent repeatedly prompts for bash write operations
- Agent uses `cat`, `echo >`, or heredocs for file creation
- Permission prompts slow down parallel agent execution

---

## Improving Agents Over Time

Use friction reports to iteratively improve:

1. **Track patterns:** Which agents hit which friction points?
2. **Evaluate tradeoffs:** Should agent get more tools, or is coordination better?
3. **Update configs:** Add tools where appropriate
4. **Document patterns:** Update this skill with new coordination patterns

### Decision Framework

```
Agent frequently needs Write access?
├── Yes, core to its function → Add Write to tools config
├── Yes, but rarely → Use coordination pattern
└── No, was a one-off → No change needed
```
