# Prompt Engineering Tips for Task Agents

Curated best practices from official Claude documentation for constructing effective task prompts.

## Official Documentation

For comprehensive guidance, consult:
- [Claude 4 Best Practices](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/claude-4-best-practices.md)
- [Prompt Engineering Overview](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview.md)

---

## Key Principles

### 1. Be Explicit with Instructions

Claude 4.x models respond well to clear, explicit instructions. Specificity enhances results.

**Less Effective:**
```
Look at the authentication code
```

**More Effective:**
```
Read src/auth/login.ts and identify the validation logic for email addresses.
List any edge cases that aren't handled.
```

### 2. Add Context/Motivation

Explaining WHY helps Claude understand goals and deliver targeted responses.

**Less Effective:**
```
Never return partial results
```

**More Effective:**
```
This audit will be used for compliance review. Returning partial results
could cause us to miss critical issues, so always complete the full analysis
even if it takes longer.
```

### 3. Be Vigilant with Examples

Claude pays close attention to examples. Ensure they align with desired behaviors.

**When providing examples:**
- Show the exact format you want
- Include edge cases if relevant
- Make sure examples are accurate

### 4. Use XML Tags for Structure

XML tags create clear sections that improve instruction following.

```xml
<context>
Background information the agent needs
</context>

<task>
The specific action to perform
</task>

<constraints>
- Limitation 1
- Limitation 2
</constraints>

<output_format>
How to structure the response
</output_format>
```

---

## Optimizing Parallel Tool Calling

Claude 4.x models excel at parallel execution. To maximize efficiency:

```
If you intend to call multiple tools and there are no dependencies between
the tool calls, make all independent calls in parallel. For example, when
reading 3 files, run 3 tool calls in parallel.

However, if some tool calls depend on previous calls, do NOT call these
tools in parallel — call them sequentially.
```

**When to parallelize:**
- Reading multiple independent files
- Searching different directories
- Running independent analyses

**When to sequence:**
- Results from one task inform another
- One task modifies what another needs to read
- Order matters for correctness

---

## State Management Best Practices

For tasks spanning multiple turns or context windows:

### Use Structured Formats for Status Data

```json
{
  "tasks": [
    {"id": 1, "name": "verify_citations", "status": "complete"},
    {"id": 2, "name": "update_claims", "status": "in_progress"},
    {"id": 3, "name": "final_review", "status": "pending"}
  ]
}
```

### Use Freeform Text for Progress Notes

```
Session progress:
- Verified 12/15 citations in chronic-pain.md
- Found 2 requiring correction (see corrections.md)
- Next: Complete remaining 3 citations, then move to exercise.md
```

### Git as State Tracking

Encourage agents to use git for checkpointing:
- Commit working changes before risky operations
- Use descriptive commit messages for context
- Reference commits when resuming work

---

## Tool Usage Patterns

### Default to Action

If you want agents to implement rather than suggest:

```
By default, implement changes rather than only suggesting them. If the
user's intent is unclear, infer the most useful likely action and proceed.
```

### Conservative Action

If you want agents to ask before acting:

```
Do not make changes unless explicitly instructed. When intent is ambiguous,
default to providing information and recommendations rather than taking action.
```

---

## Avoiding Common Issues

### Reduce Vague Instructions

| Vague | Explicit |
|-------|----------|
| "Check the code" | "Read auth.ts and verify input validation handles empty strings" |
| "Fix the bug" | "The error occurs when username contains spaces. Update the regex in validateUsername()" |
| "Make it better" | "Refactor to extract the validation logic into a separate function" |

### Ensure Output Format Clarity

Always specify what you want returned:

```
<output_format>
Return a markdown report with:
1. Summary section (2-3 sentences)
2. Findings table (columns: Item, Status, Notes)
3. Recommended actions (numbered list)
</output_format>
```

### Prevent Overengineering

If agents tend to add unnecessary complexity:

```
Avoid over-engineering. Only make changes that are directly requested.
Keep solutions simple and focused. Don't add features, refactor code,
or make "improvements" beyond what was asked.
```

---

## Long-Horizon Task Tips

For complex tasks that may span multiple turns:

1. **Break into phases** — Research, Design, Implement, Verify
2. **Save state explicitly** — Write progress to files
3. **Use incremental commits** — Checkpoint working states
4. **Verify after each phase** — Don't accumulate errors

```
This is a multi-step task. After each phase:
1. Summarize what was accomplished
2. Note any issues or blockers
3. State what the next phase should address
```

---

## Quick Reference

**Always include:**
- Clear task description (what to do)
- Relevant context (file paths, constraints)
- Output format (how to structure response)
- Skill invocation syntax (if using a skill)

**Avoid:**
- Vague instructions ("look into this")
- Missing output format
- Implicit assumptions about what tools to use
- Parallelizing dependent tasks
