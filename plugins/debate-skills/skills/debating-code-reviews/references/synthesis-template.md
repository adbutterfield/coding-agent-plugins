---
title: Synthesis Template
purpose: Final report format and triage criteria for debate review results
---

# Synthesis Template

Template for the lead to produce a final consolidated report after the debate rounds complete.

## Table of Contents

1. [Report Structure](#report-structure) - Complete markdown template
2. [Triage Criteria](#triage-criteria) - When to mark CONFIRMED / DISPUTED / DISMISSED
3. [Overall Verdict](#overall-verdict) - APPROVE / APPROVE WITH CHANGES / REQUEST CHANGES

---

## Report Structure

```markdown
# Debate Review: [PR/Change Title]

**Reviewers:** Tracer, Architect, Breaker, Prosecutor
**Rounds completed:** 3
**Date:** [date]
**Scope:** [files/functions reviewed]

---

## Confirmed Findings

> High confidence - Multiple reviewers agree or Prosecutor verified.
> These should be addressed before merge.

### [BLOCKING] Finding title
- **Found by:** [reviewer name]
- **Confirmed by:** [other reviewers / Prosecutor]
- **Description:** [clear description of the issue]
- **Evidence:** [execution path, code quotes, or test scenario]
- **Suggested fix:** [if reviewers proposed one]

### [WARNING] Finding title
- **Found by:** [reviewer name]
- **Description:** [clear description]
- **Evidence:** [supporting details]
- **Suggested fix:** [if applicable]

---

## Disputed Findings

> Reviewers disagreed - Needs human judgment.
> Include both positions so the author can decide.

### Finding title
- **Position A** ([reviewer name]): [their argument]
- **Position B** ([reviewer name]): [their counter-argument]
- **Prosecutor's take:** [verdict and reasoning]
- **Recommendation:** [lead's suggested resolution, if any]

---

## Dismissed Findings

> Low confidence - Prosecutor challenged successfully or reviewer retracted.
> Listed for transparency, no action required.

### Finding title
- **Originally raised by:** [reviewer name] as [original severity]
- **Dismissed because:** [reason - false positive, overstated, lacks evidence]

---

## Test Quality Assessment

> From The Breaker's test review.

### Coverage
- **Well-tested areas:** [list]
- **Gaps identified:** [list of untested scenarios]

### Test Quality Issues
- [Any assertion-free tests, fragile tests, or implementation-coupled tests]

### Recommendations
- [Specific tests to add or improve]

---

## Debate Highlights

> Most interesting disagreements or discoveries from the rounds.

1. [Brief description of a notable debate moment and its resolution]
2. [Another notable moment]

---

## Summary

| Severity | Count | Action |
|----------|-------|--------|
| BLOCKING | N | Must fix |
| WARNING | N | Should fix |
| NOTE | N | Consider |
| Dismissed | N | No action |

**Overall verdict:** [APPROVE / APPROVE WITH CHANGES / REQUEST CHANGES]
```

---

## Triage Criteria

### When to mark CONFIRMED
- At least 2 reviewers independently identified the same issue
- Prosecutor verified the evidence holds up
- A single reviewer found it BUT provided concrete proof (execution trace, failing test, specific input)

### When to mark DISPUTED
- Reviewers disagree on severity (one says BLOCKING, another says NOTE)
- Prosecutor challenged and reviewer rebutted, but neither fully proved their case
- The finding depends on context the reviewers don't have (e.g., "how is this function called in production?")

### When to mark DISMISSED
- Prosecutor showed the finding is a false positive (the claimed behavior can't actually occur)
- Reviewer retracted after seeing additional context
- Finding is about a pre-existing issue unrelated to the change under review
- Severity is based on a theoretical scenario that requires implausible conditions

### Overall Verdict

| Verdict | Criteria |
|---------|----------|
| **APPROVE** | No BLOCKING findings, warnings are minor |
| **APPROVE WITH CHANGES** | No BLOCKING findings, but warnings worth addressing |
| **REQUEST CHANGES** | One or more BLOCKING findings confirmed |
