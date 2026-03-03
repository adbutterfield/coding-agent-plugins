---
title: Plan Template
purpose: Output format for the final plan document produced by the planning debate
---

# Plan Template

Template for the lead to produce a final consolidated plan document after the debate rounds complete. This format is designed to be directly consumable by a coding agent.

## Table of Contents

1. [Plan Document Structure](#plan-document-structure) - Complete markdown template
2. [Section Guidance](#section-guidance) - What to include in each section
3. [Quality Checklist](#quality-checklist) - Verify before handing to a coding agent

---

## Plan Document Structure

```markdown
# Implementation Plan: [Task Title]

**Task:** [one-line description]
**Debated by:** Scout, Planner, Critic, Prosecutor
**Rounds completed:** [number]
**Date:** [date]

---

## Discovery Summary

> From The Scout, verified by The Prosecutor.

### Existing Patterns to Follow
| Pattern | File | Relevance |
|---------|------|-----------|
| [name] | [path] | [why follow this] |

### Reusable Utilities
| Utility | File | Usage |
|---------|------|-------|
| [name] | [path] | [how to use it] |

### Affected Systems
| System | File | Impact |
|--------|------|--------|
| [name] | [path] | [what changes] |

### Type/Interface Constraints
- [type at path - what must be satisfied]

---

## Implementation Plan

> From The Planner, revised after Scout's discoveries, Critic's risks,
> and Prosecutor's challenges.

### Approach
[2-3 sentences: what approach and why]

### Alternatives Considered
| Approach | Why Rejected |
|----------|-------------|
| [name] | [specific reason] |

### Steps

1. **[Action] `[file path]`**
   - Changes: [specific changes]
   - Complexity: LOW / MEDIUM / HIGH
   - Depends on: [step numbers, if any]

2. **[Action] `[file path]`**
   - Changes: [specific changes]
   - Complexity: LOW / MEDIUM / HIGH

[continue for all steps...]

### File Manifest
| File | Action | Description |
|------|--------|-------------|
| [path] | create / modify / delete | [what changes] |

---

## Risk Mitigations

> From The Critic, verified by The Prosecutor.

### Confirmed Risks
| Risk | Severity | Mitigation | Addressed in Step |
|------|----------|------------|-------------------|
| [risk] | HIGH / MEDIUM | [how to prevent] | [step number] |

### Disputed Risks (Needs Human Judgment)
| Risk | Critic's Position | Prosecutor's Position |
|------|-------------------|----------------------|
| [risk] | [argument] | [counter-argument] |

### Dismissed Risks
| Risk | Dismissed Because |
|------|-------------------|
| [risk] | [reason - false positive, already handled, can't occur] |

---

## Test Strategy

> Grounded in existing test patterns found by The Scout.

### Test Framework
[framework name and relevant config]

### Tests to Write
| Test | Type | Validates |
|------|------|-----------|
| [description] | unit / integration / e2e | [what it proves] |

### Edge Cases to Cover
- [edge case from Critic's analysis]

---

## Open Questions

> Items that need human input before implementation begins.

- [ ] [question - what the debate couldn't resolve]
```

---

## Section Guidance

### Discovery Summary
- Only include discoveries that are **relevant to the implementation**. Don't dump everything the Scout found.
- Every discovery should be **verified by the Prosecutor** (CONFIRMED status). Mark any DISPUTED discoveries.
- Include file paths so the coding agent can navigate directly to the relevant code.

### Implementation Plan
- Steps must be **specific enough to execute without questions**. "Add validation" is too vague. "Add a `validateEmail(input: string): boolean` function to `src/utils/validation.ts` that checks format with regex and MX record existence" is specific enough.
- The plan should reflect the **revised** version from Round 3, not the Planner's original Round 1 draft.
- Complexity ratings help the coding agent estimate effort and identify steps that need extra care.

### Risk Mitigations
- Only include risks that **survived the Prosecutor's challenge** (CONFIRMED or DISPUTED).
- Each confirmed risk must have a mitigation that maps to a specific step in the plan.
- Disputed risks are presented with both positions so the human can decide.

### Test Strategy
- Base the testing approach on **existing patterns** the Scout found, not ideal testing practices.
- Include edge cases identified by the Critic.

### Open Questions
- These are items the debate could not resolve and need human input.
- The coding agent should NOT start implementation until these are answered.

---

## Quality Checklist

Before handing the plan to a coding agent, verify:

- [ ] Every file in the file manifest has at least one corresponding step
- [ ] Every step specifies a concrete action (not "update" or "modify" without details)
- [ ] All file paths are real paths verified against the codebase
- [ ] Confirmed risks each have a mitigation mapped to a plan step
- [ ] The test strategy matches the project's actual testing patterns
- [ ] Open questions are clearly flagged as blockers
- [ ] The approach section explains why this approach over alternatives
- [ ] Dependencies between steps are documented
