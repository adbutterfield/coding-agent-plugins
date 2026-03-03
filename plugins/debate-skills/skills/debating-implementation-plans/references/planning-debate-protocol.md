---
title: Planning Debate Protocol
purpose: Round structure, messaging patterns, and coordination for adversarial implementation planning debates
---

# Planning Debate Protocol

Structured protocol for orchestrating multi-agent adversarial planning debates using Claude Code agent teams.

## Table of Contents

1. [Agent Team Mechanics](#agent-team-mechanics) - Task list, plan approval, messaging, context
2. [Round Structure](#round-structure) - Round 1 (parallel), Round 2 (challenge), Round 3 (final)
3. [Why Round 1 Is Parallel](#why-round-1-is-parallel) - Design rationale
4. [Risk Severity Definitions](#risk-severity-definitions) - HIGH / MEDIUM / LOW
5. [Confidence Levels](#confidence-levels) - HIGH / MEDIUM / LOW
6. [Timing Guidance](#timing-guidance) - Expected durations per round
7. [Quality Gate Hooks](#optional-quality-gate-hooks) - TeammateIdle, TaskCompleted
8. [Cleanup](#cleanup) - Shutting down the team
9. [Anti-patterns to Avoid](#anti-patterns-to-avoid) - Common mistakes

---

## Agent Team Mechanics

This protocol uses Claude Code agent teams, where each planning agent is a separate Claude Code instance with its own context window. Key mechanics:

- **Shared task list** - Rounds are modeled as tasks with dependencies. Round 2 tasks depend on Round 1 tasks, so they auto-unblock when Round 1 completes.
- **Plan approval** - The Prosecutor uses plan approval mode. The lead doesn't approve its plan until Round 1 outputs are available.
- **Messaging** - Use `message` for targeted communication (lead to one teammate). Use `broadcast` sparingly (sends to all teammates, token costs scale with team size).
- **Context** - Teammates load project context (CLAUDE.md, skills, MCP servers) but do NOT inherit the lead's conversation history. The spawn prompt must include the full task specification.

---

## Round Structure

### Round 1: Independent Analysis (Parallel)

**Participants:** Scout, Planner, Critic (3 agents)
**Prosecutor:** In plan approval mode - waiting for Round 1 to complete

Each agent claims their Round 1 task from the shared task list and independently analyzes the task specification.

**Output format per agent:**

```markdown
## [Agent Name] - Round 1 Output

### [Section based on agent role]
[Structured findings following the agent's output format]
```

**Coordination:** All three agents run in parallel via the shared task list. They work independently and do not see each other's outputs until Round 2.

---

### Round 2: Challenge and Cross-pollination (Sequential)

Round 2 tasks auto-unblock when all Round 1 tasks are marked complete.

**Step 1: Share outputs**
The lead broadcasts all Round 1 outputs to all teammates. This is one of the few times broadcast is appropriate - all teammates need the same information.

**Step 2: Prosecutor reviews**
The lead approves the Prosecutor's plan. The Prosecutor examines all Round 1 outputs and provides verdicts for each claim (CONFIRMED / DISPUTED / DISMISSED).

**Step 3: Agents respond**
The lead messages Scout, Planner, and Critic individually with the Prosecutor's challenges and each other's outputs. Use targeted `message` rather than `broadcast` since each agent gets a tailored prompt:

- **To Planner:** "Here are the Scout's discoveries you may have missed: [...]. Here are the Critic's risks your plan doesn't address: [...]. Here are the Prosecutor's challenges to your assumptions: [...]. Revise your approach."
- **To Critic:** "Here is the Planner's actual plan: [...]. Given this concrete plan, are your risks still valid? Any new risks? Here are the Prosecutor's challenges to your risks: [...]."
- **To Scout:** "Here is the Planner's plan: [...]. Did you miss any exploration areas that the plan requires? Are there utilities or patterns the Planner should know about that you didn't find in Round 1?"

**Messaging pattern:**

```
Lead → broadcast: "Round 1 complete. Here are all outputs: [consolidated]"
Lead → approve Prosecutor's plan
Prosecutor → Lead: "[Verdicts for each claim]"
Lead → message Planner: "Scout's discoveries, Critic's risks, Prosecutor's challenges: [...]"
Lead → message Critic: "Planner's plan, Prosecutor's challenges to your risks: [...]"
Lead → message Scout: "Planner's plan. Missing exploration areas? [...]"
Each agent → Lead: "[Responses, revisions, new findings]"
```

---

### Round 3: Final Outputs

Each agent produces their final deliverable:

- **Planner:** Revised implementation plan incorporating Scout's discoveries, Critic's risks, and Prosecutor's challenges
- **Critic:** Final risk assessment - confirmed risks with mitigations, dismissed risks, risks needing human judgment
- **Prosecutor:** Final verdicts - which claims survived challenge, which remain disputed

The Scout typically does not need a Round 3 output unless Round 2 revealed missing exploration areas.

---

## Why Round 1 Is Parallel

The three Round 1 agents run simultaneously, not sequentially (Scout → Planner → Critic). This is a deliberate design choice:

1. **The delta is the value.** The Planner does its own limited exploration and produces a draft plan. The Scout does exhaustive exploration. The difference between what the Planner found on its own vs what the Scout found is exactly the kind of gap that causes implementation failures. If the Planner had the Scout's findings first, this gap would be invisible.

2. **The Critic doesn't need the plan.** Pre-mortem analysis works from the task specification, not the solution. "What could go wrong with implementing X?" is answerable without knowing the specific approach. In Round 2, the Critic gets the plan and can then identify specific vulnerabilities.

3. **Structural consistency.** This matches the review skill's Round 1 structure where Tracer, Architect, and Breaker all analyze independently before the Prosecutor challenges.

---

## Risk Severity Definitions

| Severity | Meaning | Action Required |
|----------|---------|-----------------|
| **HIGH** | Will cause implementation failure, major rework, or broken functionality if not addressed in the plan | Must address before implementation begins |
| **MEDIUM** | May cause delays, suboptimal implementation, or minor rework | Should address, may proceed with documented risk |
| **LOW** | Minor concern or optimization opportunity | Note for implementer, no plan changes required |

---

## Confidence Levels

| Confidence | Meaning |
|------------|---------|
| **HIGH** | Multiple agents agree, evidence verified against actual code, or Prosecutor confirmed |
| **MEDIUM** | Single agent found it, evidence is reasonable but not independently verified |
| **LOW** | Theoretical concern, based on assumptions, or Prosecutor disputed |

---

## Timing Guidance

| Round | Expected Duration | Notes |
|-------|-------------------|-------|
| Round 1 | 3-5 minutes | Parallel, wall time equals longest agent (usually Scout) |
| Round 2 | 5-8 minutes | Sequential: Prosecutor first, then responses |
| Round 3 | 3-5 minutes | Planner produces revised plan, others provide final positions |
| Synthesis | 2-3 minutes | Lead consolidates into plan document |
| **Total** | **13-21 minutes** | For a complex implementation task |

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
            "command": "echo 'Verify output includes evidence from actual codebase, not assumptions'"
          }
        ]
      }
    ]
  }
}
```

- **`TaskCompleted`** - Runs when a task is marked complete. Exit with code 2 to prevent completion and send feedback.
- **`TeammateIdle`** - Runs when a teammate is about to go idle. Exit with code 2 to send feedback and keep the teammate working.

---

## Cleanup

After synthesis, ask the lead to shut down teammates and clean up:

```
Shut down all teammates, then clean up the team.
```

Always shut down teammates before cleanup. The lead must run cleanup - teammates should not.

---

## Anti-patterns to Avoid

1. **Sequential Round 1** - Don't run Scout → Planner → Critic sequentially. The independent-then-compare pattern is the whole point. If the Planner sees the Scout's findings first, you lose the valuable delta.
2. **Skipping the Scout** - "I'll just plan it" leads to reinventing existing utilities and missing affected systems. Even for medium-complexity tasks, at minimum run the Scout.
3. **Vague plans** - The Planner must produce specific steps with file paths and concrete changes. "Update the service layer" is not a plan step.
4. **Theoretical risks** - The Critic must ground risks in evidence from the actual codebase. "Performance might be an issue" without evidence is not useful.
5. **Prosecutor overreach** - The Prosecutor challenges claims, not creates plans. It should not introduce new implementation ideas.
6. **Broadcast overuse** - Use `broadcast` only when all teammates need the same information (sharing Round 1 outputs). Use targeted `message` for agent-specific prompts.
7. **Missing task context** - Teammates don't inherit the lead's conversation history. Always include the full task specification in the spawn prompt.
8. **Ignoring the plan template** - The final output must follow the plan template format so coding agents can consume it directly.
