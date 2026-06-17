# FAQ

## When should I use Strategy-First Executor vs. a simple prompt?

**Use Strategy-First Executor when:**
- Task has 3+ distinct phases with clear artifact boundaries
- Task may be interrupted (long-running, need to resume later)
- You need an audit trail of what happened
- Quality matters and you want verifiable checkpoints
- Previous attempts showed LLM context drift

**Use a simple prompt when:**
- Task is a single step (one search, one file write)
- No need for resumability
- You're exploring/experimenting, not executing a known workflow

## What happens if the system crashes mid-phase?

The Recovery Protocol (ADR-006) kicks in on restart:
1. Scans `journal.jsonl` for last `phase_completed` event → safe anchor
2. Counts `checkpoint_failed` events → restores retry_count
3. Ignores partial artifacts from interrupted phase → they'll be regenerated
4. Resumes from Consume Verification of target phase

You lose at most one phase of work. No manual cleanup needed.

## How many replans are allowed?

Default: `max_replan: 2`. You can adjust in `policy.yaml`.

Each replan requires a `[REPLAN ANALYSIS]` block explaining why the previous strategy failed and what changed. If the task is fundamentally impossible, the strategy prompt can declare this instead of generating another doomed strategy.

## Can I skip the checkpoint gate?

No. Consume Verification (Step 1 of checkpoint gate) is mandatory per ADR-005. Every phase must verify its inputs exist before executing.

## How do I add a new constraint type?

Constraint types are in the DSL (metric, semantic, script, regex, tool). If you need a new type:
1. Propose it as a new `type` value (not a new primitive — see ADR-003)
2. Define its verification protocol in the checkpoint gate
3. Add an example to `examples/strategies/`

## What's the difference between retry and replan?

- **Retry:** Constraint failed, but the strategy is still valid. Fix the artifact and re-run the checkpoint gate. Counted per-phase.
- **Replan:** Strategy itself is wrong. Generate a new strategy with failure analysis. Counted per-run.

## Can I run multiple tasks concurrently?

Each Run has its own `.runs/<run_id>/` directory with isolated artifacts and journal. Multiple runs can execute concurrently without interference — they're independent instances.

## Does this work with any OpenClaw tool?

Yes. The runtime doesn't restrict which tools phases can use. The `actions` field in strategy.yaml can reference any OpenClaw tool. Constraint type `tool` can also invoke any OpenClaw tool for verification.

## How do I version-control a strategy?

`strategy.yaml` is a plain YAML file — commit it to git. The `[REPLAN ANALYSIS]` block in replanned strategies serves as a natural changelog.

## What's the StraTA connection?

The Strategy-First Executor is directly inspired by the StraTA framework ([arxiv 2605.06642](https://arxiv.org/abs/2605.06642)). The core concepts — strategy-first decomposition, typed constraints, event-sourced execution, policy-bounded runtime — all come from StraTA. The ADRs document where we've extended or diverged from the paper.
