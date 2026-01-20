---
name: developing-react-apps
description: Use this skill when writing, reviewing, or optimizing React and Next.js code. Provides 47 performance best practices covering async patterns, bundle optimization, server/client rendering, re-render prevention, and JavaScript performance. Triggers on tasks involving React components, hooks, data fetching, code splitting, memoization, or performance improvements.
license: MIT
---

# React Performance Best Practices

47 performance optimization rules for React and Next.js applications, organized by impact.

## When to Apply

Reference these guidelines when:

- Writing new React components or Next.js pages
- Implementing data fetching (client or server-side)
- Reviewing code for performance issues
- Optimizing bundle size or load times

## Rule Categories

| Priority | Category | Impact |
|----------|----------|--------|
| 1 | Eliminating Waterfalls | CRITICAL |
| 2 | Bundle Size Optimization | CRITICAL |
| 3 | Server-Side Performance | HIGH |
| 4 | Client-Side Data Fetching | MEDIUM-HIGH |
| 5 | Re-render Optimization | MEDIUM |
| 6 | Rendering Performance | MEDIUM |
| 7 | JavaScript Performance | LOW-MEDIUM |
| 8 | Advanced Patterns | LOW |

## Finding Rules

See [references/REFERENCE.md](references/REFERENCE.md) for the complete rule index with descriptions and keywords.

**Searching for relevant rules:** Use the Keywords column in REFERENCE.md to grep for patterns like `await`, `useCallback`, `localStorage`, etc.

## Rule Format

Each rule file follows this structure:

```markdown
---
title: Rule Name
impact: CRITICAL/HIGH/MEDIUM/LOW
---

## Rule Name

> **Review:** Pattern to look for in existing code
> **Writing:** Situation when writing new code
> **Technique:** Transformation or solution to apply

Description with impact details.

### Fail
[Code example with ❌ comment explaining the problem]

### Pass
[Code example with ✅ comment explaining why it's better]

### Why This Matters
[1-3 bullet points on core principles]

### When to Apply
[Contexts where this rule is relevant]

### When to Skip
[Edge cases where the rule shouldn't apply]

### See Also
[Related rules]
```

### Using Rules for Code Review

1. Scan REFERENCE.md Keywords column for patterns in the code under review
2. Read matched rule's **Review** line for the specific anti-pattern
3. Check **When to Skip** before flagging—some patterns are intentional
4. Reference the **Pass** example when suggesting fixes

### Using Rules When Writing Code

1. When facing a situation (data fetching, state updates, etc.), scan Keywords for relevant terms
2. Read the **Writing** line to confirm the rule applies to your situation
3. Apply the **Technique** using the **Pass** example as a template
4. Check **When to Skip** to ensure the optimization is warranted
