---
name: orchestrating-task-agents
description: Use this skill when launching Task agents, batch processing multiple files, delegating work to sub-agents, or running parallel operations. Provides XML prompt structure, agent type selection, and coordination patterns.
---

# Orchestrating Task Agents

Best practices for the main agent when delegating work to Task agents (sub-agents).

## When to Use Subagents vs Main Conversation

**Keep in main conversation when:**
- Task needs frequent back-and-forth or iteration
- Multiple phases share significant context
- Making quick, targeted changes
- Latency matters (subagents start fresh, need time to gather context)

**Delegate to subagent when:**
- Task produces verbose output (test runs, log analysis, doc fetches)
- Work is self-contained and can return a summary
- Want to preserve main conversation context for other work
- Running parallel independent investigations

**Consult this skill when:**
- About to launch 3+ Task agents
- Batch processing files (transform, validate, migrate)
- Sub-agents need to invoke skills (requires explicit Skill tool syntax — they won't invoke automatically)
- Coordinating research → plan → implement phases
- Uncertain about agent type selection

## Core Principles

1. **Explicit over implicit** — Task agents need exact instructions, not hints
2. **Context is critical** — Include file paths, constraints, and expectations
3. **Parallel when possible** — Launch independent tasks in a single message
4. **Right agent for the job** — Choose the appropriate agent type

## Quick Reference

**Prompt structure:**
- `<context>` — Background, file paths, constraints
- `<task>` — What to accomplish
- `<output_format>` — How to structure results

**Agent types:**
- `Explore` — Research, file searches (default for read-only)
- `Plan` — Design implementation before coding
- `Bash` — Terminal commands only
- `general-purpose` — Implementation requiring write access

**Models:**
- `haiku` — Fastest (Explore default)
- `sonnet` — Balanced
- `opus` — Most capable
- `inherit` — Match main conversation

**Parallel:** Single message with multiple Task tool calls
**Sequential:** Wait for results before dependent tasks

## Skill Invocation Syntax (CRITICAL)

Task agents don't automatically invoke skills. You must provide explicit Skill tool syntax.

### What DOESN'T Work

```
"Use the auditing-citations skill to verify these claims"
```

The task agent receives this instruction but doesn't know to call the Skill tool.

### What WORKS

```
Verify these citations.

To access the citation verification workflow, invoke the Skill tool:
  Skill(skill: 'auditing-citations')

The skill provides structured verification templates and red flag criteria.
```

### Template for Skill Invocation

When delegating a task that should use a skill, include this pattern:

```
<task>
[Description of what the agent should accomplish]
</task>

<skill_invocation>
Invoke the [skill-name] skill using the Skill tool:
  skill: "[skill-name]"
  args: "[optional arguments if needed]"

This skill provides: [brief description of what the skill offers]
</skill_invocation>

<output_format>
[What the agent should return]
</output_format>
```

## Task Prompt Structure

Use XML tags for clear structure:

```xml
<context>
[Background information, file paths, constraints]
</context>

<task>
[Explicit instructions - what exactly to do]
</task>

<output_format>
[How to structure the response]
</output_format>
```

### Be Explicit, Not Vague

| Vague (Less Effective) | Explicit (More Effective) |
|------------------------|---------------------------|
| "Look at the code" | "Read src/auth/login.ts and identify the validation logic" |
| "Fix the bug" | "The login fails when email contains '+'. Fix the regex in validateEmail()" |
| "Use the skill" | "Invoke the Skill tool: Skill(skill: 'skill-name')" |
| "Improve this" | "Add error handling for network timeouts in fetchUser()" |

### Add Context/Motivation

Explain WHY the task matters:

```
<context>
We're auditing citations before product launch. Accuracy is critical because
incorrect claims could violate FTC guidelines.
</context>

<task>
Verify the research citations in articles/chronic-pain.md
</task>
```

## Agent Type Selection

| Agent Type | Use When | Tools Available |
|------------|----------|-----------------|
| `Explore` | Quick codebase searches, finding files, understanding patterns | Read-only tools |
| `Plan` | Designing implementation, architectural decisions | Read-only tools |
| `Bash` | Git operations, running commands, terminal tasks | Bash only |
| `general-purpose` | Complex multi-step work requiring all capabilities | All tools |

### Custom Agents (agents/ directory)

User-defined agents have their own tool configurations. If an agent lacks Write/Edit, it may use bash workarounds that trigger permission prompts.

See `./references/custom-agent-coordination.md` for:
- Tool access verification patterns
- Coordination pattern (agent returns content → main agent writes)
- Post-task feedback loops for improving agent configs

### Selection Guidelines

- **Default to `Explore`** for research and file searches
- **Use `Plan`** when you need implementation design before coding
- **Use `Bash`** for isolated command execution
- **Use `general-purpose`** only when the task requires writing/editing files

## Model Selection

When launching Task agents, you can specify `model`:

| Model | Characteristics |
|-------|-----------------|
| `haiku` | Fastest, good for simple searches |
| `sonnet` | Balanced capability and speed |
| `opus` | Most capable, best for complex reasoning |
| `inherit` | Match main conversation model |

**If cost matters:** Use `haiku` for simple searches, `sonnet` for implementation, `opus` for complex analysis.

**If cost doesn't matter (e.g., Claude Max):** Default to `opus` for best results across all tasks. Speed is the only tradeoff.

**Note:** Explore agents default to `haiku`. Override with `model: opus` if you want more nuanced analysis.

## Coordination Patterns

### Parallel Tasks (Independent)

Launch multiple agents in a single message when tasks don't depend on each other:

```
[Task 1]: Explore agent to find authentication patterns
[Task 2]: Explore agent to find API endpoint structure
[Task 3]: Explore agent to find test patterns
```

All three run simultaneously, maximizing efficiency.

### Sequential Tasks (Dependent)

Wait for results when later tasks depend on earlier ones:

```
Step 1: Explore agent finds relevant files
Step 2: (After results) Plan agent designs implementation
Step 3: (After approval) General-purpose agent implements
```

### Fan-Out Pattern

When the same operation applies to multiple items:

```
For each of these 3 files, launch an Explore agent:
- src/auth/login.ts
- src/auth/register.ts
- src/auth/reset.ts

Each agent analyzes validation patterns in its assigned file.
```

### Background vs Foreground Execution

**Foreground (default):** Blocks main conversation, can prompt for permissions.

**Background:** Runs concurrently. Important limitations:
- Auto-denies permissions not pre-approved
- MCP tools unavailable
- Can resume failed agents in foreground later

**When to run in background:**
- Long-running analysis you don't need immediately
- Tasks where permissions are already approved
- Parallel research that shouldn't block other work

**Tip:** Press Ctrl+B to background a running task.

### Resuming Agents

Each Task invocation creates fresh context by default. To continue previous work, ask to resume:

```
Continue the previous code review and now analyze authorization logic
```

Resumed agents retain full context (tool calls, results, reasoning). Useful for:
- Extending analysis based on initial findings
- Retrying failed background agents in foreground
- Multi-turn delegation where context matters

## Context Passing

### What to Include

- File paths (absolute or relative to project root)
- Relevant code snippets or function names
- Constraints and requirements
- Expected output format
- Any skills the agent should invoke

### What to Omit

- Full conversation history (agents have context awareness)
- Redundant explanations
- Unrelated background information

## Common Mistakes

### 1. Forgetting Skill Invocation Syntax

**Wrong:** "Use the citation-audit skill"

**Right:** "Invoke the Skill tool: Skill(skill: 'auditing-citations')"

### 2. Over-Parallelizing Dependent Tasks

**Wrong:** Launching Plan and Implementation agents simultaneously

**Right:** Wait for Plan results before launching Implementation agent

### 3. Vague Instructions

**Wrong:** "Look into the auth system"

**Right:** "Read src/auth/*.ts and identify how session tokens are validated"

### 4. Missing Output Format

**Wrong:** "Analyze the code"

**Right:** "Analyze the code and return a markdown table with columns: Function, Purpose, Concerns"

### 5. Wrong Agent Type

**Wrong:** Using `general-purpose` for simple file searches

**Right:** Using `Explore` for searches, reserving `general-purpose` for implementation work

### 6. Task Agents Using Bash for File Operations

Task agents sometimes use `cat` heredocs instead of Write tool, triggering permission prompts.

**Mitigations:**
- For custom agents: see `./references/custom-agent-coordination.md`
- For built-in `general-purpose` agents: explicitly instruct "Use the Write tool, not bash commands"

### 7. Command Variations Triggering Re-Approval

Minor command variations (like adding `2>/dev/null`) don't match previously approved commands, causing repeated permission prompts.

**Mitigations:**
- Specify exact command patterns in task prompts when consistency matters
- Have main agent run commands that need specific approval patterns

### 8. Expecting Subagents to Spawn Subagents

**Limitation:** Subagents cannot spawn other subagents.

**Wrong:** Expecting a task agent to delegate subtasks to other agents

**Right:** Chain from main conversation:
1. Agent 1 completes → returns results
2. Main agent launches Agent 2 with Agent 1's results
3. Repeat as needed

## Reference Files

- `./references/task-prompt-templates.md` — Copy-paste ready templates for common scenarios
- `./references/custom-agent-coordination.md` — Patterns for coordinating user-defined agents (tool access, feedback loops)
- `./references/prompt-engineering-tips.md` — Curated tips from official Claude documentation
