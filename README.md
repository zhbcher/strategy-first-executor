# Strategy-First Executor

An OpenClaw skill that wraps complex multi-step tasks with explicit strategy-first execution and optional self-review.

Inspired by the [StraTA](https://arxiv.org/abs/2605.06642) framework (Strategic Trajectory Abstraction).

## What It Does

Instead of reactive step-by-step execution, this skill forces the agent to:

1. **Generate a strategy** — analyze the task and output a concrete plan (goal, constraints, phases, checkpoints)
2. **Execute with guidance** — every action step is conditioned on the strategy, preventing drift
3. **Self-review** — after execution, the agent reviews its own work against the strategy and flags deviations

## When to Use

- Tasks with 3+ distinct phases (research → analyze → output)
- Multi-sub-agent coordination
- Long-horizon tasks where agents tend to lose context
- High-stakes tasks where errors are expensive

## Installation

Copy this skill into your OpenClaw workspace:

```bash
cp -r strategy-first-executor/ ~/.openclaw/workspace/skills/
```

## Structure

```
strategy-first-executor/
├── SKILL.md                          # Skill definition and workflow
├── README.md                         # This file
├── references/
│   ├── strategy-prompt.md            # Strategy generation prompt template
│   └── self-review-prompt.md         # Post-execution self-review prompt template
└── examples/
    └── video-production.md           # Real-world example: video production workflow
```

## The Strategy Is NOT Optional

Unlike a free-form plan, the strategy is:
- **Concrete** — every action can be checked against it
- **Fixed in context** — prepended to every step, never buried in history
- **Self-reviewed** — deviations are caught before the user sees them

## Reference

- Paper: [StraTA: Incentivizing Agentic Reinforcement Learning with Strategic Trajectory Abstraction](https://arxiv.org/abs/2605.06642)
- Code: [github.com/xxyQwQ/StraTA](https://github.com/xxyQwQ/StraTA)
