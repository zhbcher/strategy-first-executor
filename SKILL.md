---
name: "strategy-first-executor"
description: "Wrap complex multi-step tasks with explicit strategy-first execution, checkpoint gating, and self-review."
user-invocable: false
disable-model-invocation: false
metadata.openclaw:
  requires: {}
---

# Strategy-First Executor

Use for complex, multi-step tasks (3+ distinct steps, long-horizon, or high-risk) where reactive step-by-step execution tends to drift. Inspired by the StraTA framework (arxiv 2605.06642).

## When to Apply

**Apply this pattern when:**
- Task has 3+ distinct phases (research → analyze → output)
- Task spans multiple sub-agent spawns that need coordination
- Previous attempts showed drift, inconsistency, or forgetting

**Skip when:**
- Single-step tasks
- Trivial tasks the model handles reliably without planning
- Tasks where the strategy is obvious from context

## Workflow

### Phase 1: Strategy Generation

Before any action, analyze the task and output a strategy using `references/strategy-prompt.md`.

Each constraint must be **verifiable** — a yes/no condition, not a guideline. Optionally specify a **verification method** (tool call, script, manual check).

### Phase 2: Strategy-Guided Execution + Checkpoint Gating

Execute the plan step by step. Before each action, note which phase this belongs to.

**After every phase marked with a checkpoint, run a checkpoint gate** using `references/checkpoint-gate-prompt.md`:
- PASS → proceed to next phase
- FAIL → fix the issue, then re-run the gate
- FAIL_STRATEGY → regenerate strategy from Phase 1

If the checkpoint has a verification method (e.g., `exec: check.py`), run it before asking the model.

Never proceed past a checkpoint that says FAIL.

### Phase 3: Final Self-Review

After all phases complete, run a final self-review using `references/self-review-prompt.md`.

## Strategy Format

```
## Strategy
### Goal
[One sentence: definition of done]

### Constraints
Each constraint is a checkable yes/no condition. Add a verification method where a tool can check it.

- [Verifiable constraint] → Verify with: [tool call or manual]
- [Verifiable constraint]

### Execution Plan
1. [Phase name] — [actions, tools, expected output] 🔒 Checkpoint: [verifiable condition]
   Verify with: [optional — exec, browser, manual]
2. [Phase name] — [actions, tools, expected output] 🔒 Checkpoint: [verifiable condition]

### Checkpoints
- After Phase N: verify [condition]
```

Phases without 🔒 run without gating. Phases with `Verify with:` get objective checks before model judgment.

## Sub-agent Integration

When spawning sub-agents for parallel work:
- Inject the strategy into each sub-agent's task definition
- Each sub-agent reports back mapped to a specific phase
- The orchestrating agent aggregates outputs against the strategy

## References

- Prompt templates: `references/strategy-prompt.md`, `references/checkpoint-gate-prompt.md`, `references/self-review-prompt.md`
- StraTA paper: arxiv.org/abs/2605.06642 — the RL training framework this inference pattern is derived from
