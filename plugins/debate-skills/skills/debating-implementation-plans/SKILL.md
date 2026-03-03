---
name: debating-implementation-plans
description: >
  Use this skill when orchestrating multi-agent adversarial planning debates
  before code is written. Activates when planning complex implementations,
  coordinating pre-implementation discovery, stress-testing implementation
  approaches, or producing validated plans for coding agents. Provides
  discovery prompts, debate protocol, and plan templates.
---

# Adversarial Debate Implementation Plans

Orchestrate a team of 4 planning specialists who independently analyze a task, then debate to produce a validated implementation plan. Adversarial debate before coding catches issues when revising a plan costs nothing, instead of after coding when fixing means rewriting.

## Why Debate Before Coding

Planning failures are expensive:

- **Incomplete discovery** - The implementer misses existing utilities and reinvents them
- **First-approach anchoring** - The first viable approach gets chosen without considering alternatives
- **Cross-system blind spots** - Changes break downstream systems nobody checked
- **Optimistic complexity** - "This is straightforward" turns into a multi-day rewrite

Debate forces independent perspectives to collide before a single line of code is written.

## Prerequisites

Agent teams are experimental and disabled by default. Enable them by adding the following to your `settings.json` or shell environment:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

## When to Use

Use the full debate protocol for:

- **Complex cross-module changes** - Touching multiple system boundaries or modules
- **New architecture** - Introducing patterns, abstractions, or systems that don't exist yet
- **Unfamiliar codebase** - Working in code you (or the AI) haven't explored deeply
- **High-risk implementations** - Changes where getting the plan wrong wastes significant effort

See "When to Skip" at the bottom for lighter alternatives.

## The Planning Team

| Agent | Role | Primary Technique | Focus |
|-------|------|-------------------|-------|
| **The Scout** | Discovery | Exhaustive codebase exploration | Existing patterns, utilities, types, affected systems, test patterns |
| **The Planner** | Design | Concrete implementation planning | Step-by-step approach, file manifest, alternatives considered |
| **The Critic** | Pre-mortem | Failure mode analysis | Missed dependencies, integration risks, specification gaps, complexity |
| **The Prosecutor** | Devil's Advocate | Challenge all claims | Verifies Scout's findings, challenges Planner's assumptions, tests Critic's concerns |

Each agent is available as a custom agent in `../../agents/` and is configured with Opus, read-only tools, and this skill preloaded. See `./references/planner-personas.md` for detailed persona rationale and thinking styles.

## Running the Debate

### Step 1: Create the Agent Team

Create an agent team with the 4 planning agents. They are pre-configured in this plugin's `agents/` directory (`scout`, `planner`, `critic`, `prosecutor-planning`).

Teammates don't inherit the lead's conversation history - they only get project context (CLAUDE.md, skills, MCP servers) plus the spawn prompt. Include the full task specification in the prompt.

```
Create an agent team to plan the implementation of [describe the task - include
the full specification, acceptance criteria, and any constraints].

Spawn 4 teammates using the scout, planner, critic, and prosecutor-planning agents.
Require plan approval for the prosecutor-planning agent (it must wait for Round 1 outputs).

Create tasks with dependencies:
- Round 1 tasks (no dependencies, run in parallel):
  - "Scout: explore codebase for [task context]" → assign to scout
  - "Planner: draft implementation plan for [task context]" → assign to planner
  - "Critic: pre-mortem analysis of [task context]" → assign to critic
- Round 2 tasks (depend on all Round 1 tasks):
  - "Prosecutor: challenge Round 1 outputs" → assign to prosecutor-planning
  - "All agents: respond to challenges and cross-pollinate findings"
- Round 3 task (depends on Round 2):
  - "Planner: produce final revised plan"
  - "Critic: state final risk assessment"
  - "Prosecutor: provide final verdicts"

After Round 3, synthesize into the final plan document.
```

### Step 2: Round 1 - Independent Analysis (Parallel)

Scout, Planner, and Critic analyze the task independently. All three run in parallel via the shared task list.

The Prosecutor is in plan approval mode during Round 1 - the lead won't approve its plan until Round 1 tasks are complete.

**Why parallel, not sequential:** The Planner's draft plan will miss things the Scout finds - that delta is where the value lives. The Critic can pre-mortem from the task description alone without needing the plan.

### Step 3: Round 2 - Challenge and Cross-pollination (Sequential)

Round 2 tasks automatically unblock when Round 1 completes (via task dependencies).

1. Lead broadcasts all Round 1 outputs to all teammates
2. Prosecutor's plan is approved - it challenges all three agents' work
3. Lead messages each agent individually with the Prosecutor's challenges and each other's outputs:
   - **Planner** sees Scout's discoveries (patterns it missed, utilities it reinvented) and Critic's risks (gaps in the draft plan)
   - **Critic** sees the actual plan (can identify specific vulnerabilities)
   - **Scout** sees the plan (can flag missing exploration areas)

See `./references/planning-debate-protocol.md` for detailed messaging patterns.

### Step 4: Round 3 - Final Outputs

- **Planner** produces a revised plan incorporating all feedback
- **Critic** states final risk assessment (confirmed risks, dismissed risks)
- **Prosecutor** provides final verdicts on all claims

### Step 5: Synthesize

The lead produces the final plan document using the template in `./references/plan-template.md`.

The plan document is designed to be directly consumable by a coding agent. It includes:
1. **Discovery Summary** - Patterns, utilities, and constraints from the Scout
2. **Implementation Plan** - Revised steps from the Planner
3. **Risk Mitigations** - Confirmed risks with mitigations from the Critic
4. **Test Strategy** - Testing approach grounded in existing conventions

## When to Skip

The full 4-agent debate is thorough but costs time and tokens. Use lighter approaches for simpler work:

| Change Type | Approach | Agents | Rounds |
|-------------|----------|--------|--------|
| Complex cross-module, new architecture | Full debate | 4 | 3 |
| Medium complexity, single module | Scout + Planner | 2 | 1 |
| Simple, well-understood | Solo Scout (discovery check) | 1 | 1 |
| Trivial (docs, config) | Skip entirely | 0 | 0 |

## Quick Reference

- **Planning agents:** `../../agents/` (scout, planner, critic, prosecutor-planning)
- **Persona rationale and thinking styles:** `./references/planner-personas.md`
- **Round structure and messaging:** `./references/planning-debate-protocol.md`
- **Final plan template:** `./references/plan-template.md`
