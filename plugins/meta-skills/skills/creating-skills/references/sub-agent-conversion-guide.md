# Converting Sub-Agents to Skills Guide

This guide covers the key differences between sub-agents and skills, and the transformation steps for conversion.

## Essential Reading

Before starting any conversion, review these official documentation sources:

- **Sub-Agents Overview**: https://platform.claude.com/docs/en/claude-code/sub-agents.md
- **Agent Skills Overview**: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview.md
- **Best Practices**: https://platform.claude.com/docs/en/agents-and-tools/agent-skills/best-practices.md

Use WebFetch to access these URLs for the latest information.

## Understanding the Differences

### Sub-Agent Configuration

Sub-agents are defined in files (in `~/.claude/agents/` or `.claude/agents/`) with YAML frontmatter:

```yaml
---
name: agent-name
description: What this agent does (for Task tool invocation)
tools: [optional tool restrictions]
model: sonnet|opus|haiku
---

Agent instructions and expertise...
```

**Key characteristics:**
- Invoked explicitly by main Claude instance via Task tool
- Operate in separate context windows
- Description explains what the agent does (for explicit selection)
- Model and tools can be specified
- Self-contained instructions

### Skill Configuration

Skills are directories with a `SKILL.md` file:

```yaml
---
name: skill-name
description: When to use this skill (triggers automatic invocation by Claude)
---

Skill instructions and expertise...
```

**Key characteristics:**
- Invoked automatically by Claude when relevant (no Task tool needed)
- Description must trigger invocation (keywords + use cases)
- No model/tools specification (inherits Claude Code capabilities)
- Can have supporting files (references, scripts, assets)
- Uses progressive disclosure

## Key Transformation Steps

### 1. Description Transformation (MOST CRITICAL)

Sub-agent descriptions explain WHAT the agent does. Skill descriptions must explain WHEN to invoke.

**Transformation Formula:**
```
Sub-Agent: "Reviews code quality and provides feedback"
Skill: "Use this skill when reviewing code for quality issues, security vulnerabilities, performance problems, or best practices violations. This includes analyzing pull requests, auditing existing code, or validating new implementations."
```

**Guidelines:**
- Write in third person
- Start with "Use this skill when..."
- Include specific trigger keywords users might say
- List concrete use cases
- Keep under 1024 characters
- Think: "What user queries should invoke this?"

### 2. Name Transformation

**Sub-Agent Names** (typically nouns or noun-phrases):
- `code-reviewer`
- `debugger`
- `data-scientist`

**Skill Names** (gerund form - verb + -ing):
- `reviewing-code` (not `code-reviewer`)
- `debugging-applications` (not `debugger`)
- `analyzing-data` (not `data-scientist`)

Verify:
- Lowercase only
- Hyphens for word separation
- Max 64 characters
- Must match directory name
- No leading/trailing/consecutive hyphens

### 3. Content Transformation

#### Preserve
- Core expertise and domain knowledge
- Step-by-step approaches
- Examples (these are valuable!)
- Best practices
- Common patterns

#### Enhance
- Add explicit validation steps
- Create separate files for detailed content in `references/`
- Add troubleshooting section
- Include completion checklist
- Emphasize CLI and Node.js tooling
- Keep SKILL.md under 500 lines

#### Remove/Transform
- **Remove**: `model:` field (not used in skills)
- **Remove**: `tools:` field (skills inherit all Claude Code capabilities)
- **Transform**: Sub-agent invocation examples → Skill invocation context
- **Transform**: Self-referential language ("I am an agent") → Direct instructions

### 4. Progressive Disclosure

Skills support multi-file structures. Consider organizing:

```
skill-name/
├── SKILL.md (core instructions, <500 lines)
├── references/
│   ├── detailed-methodology.md (background theory)
│   └── review-checklist.md (detailed checklists)
├── scripts/
│   └── analyze-complexity.js (Node.js, not Python!)
└── assets/
    └── sample-data.json
```

Reference supporting files with relative paths in SKILL.md:
- `./references/detailed-methodology.md`
- `./references/review-checklist.md`

## Next Steps

- See `./sub-agent-conversion-examples.md` for detailed before/after examples
- See `./sub-agent-conversion-checklist.md` for conversion checklist and troubleshooting
