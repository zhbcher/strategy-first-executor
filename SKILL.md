---
name: "strategy-first-executor"
description: "Wrap complex multi-step tasks with explicit strategy-first execution, checkpoint gating, and self-review."
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

Each constraint must be **verifiable** — a yes/no condition, not a guideline. See strategy format below.

### Phase 2: Strategy-Guided Execution + Checkpoint Gating

Execute the plan step by step. Before each action, note which phase this belongs to.

**After every phase marked with a checkpoint in the strategy, run a checkpoint gate** using `references/checkpoint-gate-prompt.md`:
- [PASS] → proceed to next phase
- [FAIL] → fix the issue, then re-run the gate
- [FAIL, strategy broken] → regenerate strategy from Phase 1

Never proceed past a checkpoint that says FAIL.

If an action fails or an intermediate result invalidates the strategy:
- Adjust the strategy in-place if the goal is unchanged
- If the strategy itself is fundamentally wrong, regenerate from Phase 1

### Phase 3: Final Self-Review

After all phases complete, run a final self-review using `references/self-review-prompt.md`.

Flag any step for manual review if it neither followed the strategy nor advanced the task.

## Strategy Format

```
## Strategy
### Goal
[One sentence: definition of done]

### Constraints
Must be checkable yes/no conditions. Bad: "be careful with durations". Good: "Phase 3 data-duration must equal TTS output value, deviation >0.1s = error".
- [Verifiable constraint]
- [Verifiable constraint]

### Execution Plan
1. [Phase name] — [actions, tools, expected output] 🔒 Checkpoint: [condition to verify]
2. [Phase name] — [actions, tools, expected output] 🔒 Checkpoint: [condition to verify]

### Checkpoints
- After Phase N: verify [specific measurable condition]
```

Phases without a checkpoint marker run without gating.

## Sub-agent Integration

When spawning sub-agents for parallel work:
- Inject the strategy into each sub-agent's task definition
- Each sub-agent reports back mapped to a specific phase
- The orchestrating agent aggregates outputs against the strategy

## References

- Prompt templates: `references/strategy-prompt.md`, `references/checkpoint-gate-prompt.md`, `references/self-review-prompt.md`
- StraTA paper: arxiv.org/abs/2605.06642 — the RL training framework this inference pattern is derived from
