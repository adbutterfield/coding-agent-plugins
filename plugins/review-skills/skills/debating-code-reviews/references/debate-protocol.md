---
title: Debate Protocol
purpose: Round structure, messaging patterns, and coordination for adversarial code review debates
---

# Debate Protocol

Structured protocol for orchestrating multi-agent adversarial code review debates using Claude Code agent teams.

## Table of Contents

1. [Agent Team Mechanics](#agent-team-mechanics) - Task list, plan approval, messaging, context
2. [Round Structure](#round-structure) - Round 1 (parallel), Round 2 (challenge), Round 3 (final)
3. [Severity Definitions](#severity-definitions) - BLOCKING / WARNING / NOTE
4. [Confidence Levels](#confidence-levels) - HIGH / MEDIUM / LOW
5. [Timing Guidance](#timing-guidance) - Expected durations per round
6. [Quality Gate Hooks](#optional-quality-gate-hooks) - TeammateIdle, TaskCompleted
7. [Cleanup](#cleanup) - Shutting down the team
8. [Anti-patterns to Avoid](#anti-patterns-to-avoid) - Common mistakes

---

## Agent Team Mechanics

This protocol uses Claude Code agent teams, where each reviewer is a separate Claude Code instance with its own context window. Key mechanics:

- **Shared task list** - Rounds are modeled as tasks with dependencies. Round 2 tasks depend on Round 1 tasks, so they auto-unblock when Round 1 completes.
- **Plan approval** - The Prosecutor uses plan approval mode. The lead doesn't approve its plan until Round 1 findings are available.
- **Messaging** - Use `message` for targeted communication (lead to one teammate). Use `broadcast` sparingly (sends to all teammates, token costs scale with team size).
- **Context** - Teammates load project context (CLAUDE.md, skills, MCP servers) but do NOT inherit the lead's conversation history. The spawn prompt must include enough detail about the code change.

---

## Round Structure

### Round 1: Independent Review (Parallel)

**Participants:** Tracer, Architect, Breaker (3 reviewers)
**Prosecutor:** In plan approval mode - waiting for Round 1 to complete

Each reviewer claims their Round 1 task from the shared task list and independently examines the code change.

**Output format per reviewer:**

```markdown
## [Reviewer Name] - Round 1 Findings

### Finding 1: [Short title]
- **Severity:** BLOCKING / WARNING / NOTE
- **Category:** [correctness / design / adversarial / test-quality]
- **Description:** [What the issue is]
- **Evidence:** [Code quotes, execution traces, or specific scenarios]
- **Impact:** [What happens if this isn't fixed]

### Finding 2: ...
```

**Coordination:** All three reviewers run in parallel via the shared task list. They work independently and do not see each other's findings until Round 2.

---

### Round 2: Challenge (Sequential)

Round 2 tasks auto-unblock when all Round 1 tasks are marked complete.

**Step 1: Share findings**
The lead broadcasts all Round 1 findings to all teammates. This is one of the few times broadcast is appropriate - all teammates need the same information.

**Step 2: Prosecutor reviews**
The lead approves the Prosecutor's plan. The Prosecutor examines every finding from Round 1 and provides verdicts:
- **CONFIRMED** - Evidence checks out, severity is appropriate
- **DISPUTED** - Evidence is weak, severity overstated, or context missing
- **DISMISSED** - False positive or practically irrelevant

**Step 3: Reviewers respond**
The lead messages Tracer, Architect, and Breaker individually with the Prosecutor's challenges and each other's findings. Use targeted `message` rather than `broadcast` since each reviewer gets a tailored prompt focusing on findings relevant to their domain.

They respond with:

- Agreements: "I agree with [reviewer] on X because..."
- Disagreements: "I disagree with [reviewer] on X because..."
- New discoveries: "Based on [reviewer]'s finding about Y, I now also see Z..."
- Severity changes: May upgrade or downgrade their own findings
- Rebuttals: Address Prosecutor's challenges with additional evidence or concede

**Messaging pattern:**

```
Lead → broadcast: "Round 1 complete. Here are all findings: [consolidated list]"
Lead → approve Prosecutor's plan
Prosecutor → Lead: "[Verdicts for each finding]"
Lead → message Tracer: "Prosecutor's challenges on your findings: [...]. Other findings to review: [...]"
Lead → message Architect: "Prosecutor's challenges on your findings: [...]. Other findings to review: [...]"
Lead → message Breaker: "Prosecutor's challenges on your findings: [...]. Other findings to review: [...]"
Each reviewer → Lead: "[Responses, rebuttals, new findings]"
```

---

### Round 3: Final Positions

Each reviewer states their final findings with:
- Updated severity (if changed from Round 1)
- Confidence level: HIGH / MEDIUM / LOW
- Whether the finding survived debate or was retracted

The Prosecutor provides a final triage:
- Which findings are **CONFIRMED** (survived challenge)
- Which are **DISPUTED** (reviewers disagree, needs human judgment)
- Which are **DISMISSED** (retracted or disproven)

---

## Severity Definitions

| Severity | Meaning | Action Required |
|----------|---------|-----------------|
| **BLOCKING** | Bug that will cause incorrect behavior, data loss, or security issue in production | Must fix before merge |
| **WARNING** | Design issue, missing edge case handling, or test gap that should be addressed | Should fix, may defer with justification |
| **NOTE** | Style concern, minor improvement opportunity, or observation | Consider fixing, no action required |

---

## Confidence Levels

| Confidence | Meaning |
|------------|---------|
| **HIGH** | Multiple reviewers agree, evidence is concrete, or Prosecutor confirmed |
| **MEDIUM** | Single reviewer found it, evidence is reasonable but not airtight |
| **LOW** | Theoretical concern, hard to verify, or Prosecutor disputed |

---

## Timing Guidance

| Round | Expected Duration | Notes |
|-------|-------------------|-------|
| Round 1 | 3-5 minutes | Parallel, so wall time equals longest reviewer |
| Round 2 | 5-8 minutes | Sequential: Prosecutor first, then responses |
| Round 3 | 2-3 minutes | Quick final positions |
| Synthesis | 2-3 minutes | Lead consolidates |
| **Total** | **12-19 minutes** | For a moderate-sized code change |

---

## Optional: Quality Gate Hooks

Agent teams support `TeammateIdle` and `TaskCompleted` hooks. These can enforce quality standards:

```json
{
  "hooks": {
    "TaskCompleted": [
      {
        "matcher": "",
        "hooks": [
          {
            "type": "command",
            "command": "echo 'Verify finding has severity, evidence, and impact before marking complete'"
          }
        ]
      }
    ]
  }
}
```

- **`TaskCompleted`** - Runs when a task is marked complete. Exit with code 2 to prevent completion and send feedback (e.g., require all findings to include evidence).
- **`TeammateIdle`** - Runs when a teammate is about to go idle. Exit with code 2 to send feedback and keep the teammate working (e.g., "you haven't reviewed all the files yet").

---

## Cleanup

After synthesis, ask the lead to shut down teammates and clean up:

```
Shut down all teammates, then clean up the team.
```

Always shut down teammates before cleanup. The lead must run cleanup - teammates should not.

---

## Anti-patterns to Avoid

1. **Groupthink** - If all reviewers agree on everything, something is wrong. Encourage dissent.
2. **Severity inflation** - Not every finding is BLOCKING. Reserve BLOCKING for genuine correctness or security issues.
3. **Scope creep** - Review the code change, not the entire codebase. Reviewers should not go hunting for pre-existing issues unless directly related.
4. **Style wars** - The Architect reviews design, not formatting. Don't debate tabs vs spaces.
5. **Echo chamber** - Reviewers should form independent opinions in Round 1 before seeing others' findings.
6. **Prosecutor overreach** - The Prosecutor challenges findings, not the code. They should not introduce new code findings.
7. **Broadcast overuse** - Use `broadcast` only when all teammates need the same information (e.g., sharing Round 1 findings). Use targeted `message` for reviewer-specific prompts.
8. **Missing spawn context** - Teammates don't inherit the lead's conversation history. Always include file paths, change description, and review scope in the spawn prompt.
