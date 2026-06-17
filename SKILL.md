---
name: "strategy-first-executor"
description: "Wrap complex multi-step tasks with explicit strategy-first execution and optional self-review."
---

# Strategy-First Executor

Use for complex, multi-step tasks (3+ distinct steps, long-horizon, or high-risk) where reactive step-by-step execution tends to drift. Inspired by the StraTA framework (arxiv 2605.06642).

## When to Apply

**Apply this pattern when:**
- Task has 3+ distinct phases (research → analyze → output)
- Task spans multiple sub-agent spawns that need coordination
- Previous attempts showed drift, inconsistency, or forgetting
- The user explicitly asks for a plan-first approach

**Skip when:**
- Single-step tasks
- Trivial tasks the model handles reliably without planning
- Tasks where the strategy is obvious from context

## Workflow

### Phase 1: Strategy Generation

Before any action, analyze the task and output a strategy using `references/strategy-prompt.md`.

### Phase 2: Strategy-Guided Execution

Execute the plan step by step. Before each action, note which phase of the strategy this belongs to.

If an action fails or an intermediate result invalidates the strategy:
- Adjust the strategy in-place (no need to restart from scratch if the goal is unchanged)
- If the strategy itself is fundamentally wrong, regenerate from Phase 1

### Phase 3: Self-Review

After execution, run a self-review against the strategy using `references/self-review-prompt.md`.

Flag any step for manual review if it neither followed the strategy nor advanced the task.

## Strategy Format

```
## Strategy
### Goal
[One sentence: definition of done]

### Constraints
- [Hard rules, format requirements, must-not-do items]

### Execution Plan
1. [Phase name] — [what to do, tools, expected output]
2. [Phase name] — [what to do, tools, expected output]

### Checkpoints
- After step N: verify [condition]
```

## Sub-agent Integration

When spawning sub-agents for parallel work:
- Inject the strategy into each sub-agent's task definition
- Each sub-agent reports back mapped to a specific phase
- The orchestrating agent aggregates outputs against the strategy

## References

- Prompt templates: `references/strategy-prompt.md`, `references/self-review-prompt.md`
- StraTA paper: arxiv.org/abs/2605.06642 — the RL training framework this inference pattern is derived from
