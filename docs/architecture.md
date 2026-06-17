# Architecture Design

## Positioning

The Strategy-First Executor is a **Task Runtime** — a layer between OpenClaw (platform) and business skills:

```
L0: OpenClaw Platform        (session, tool, memory, channel)
L1: Task Runtime             (strategy, execution, verification) ← this project
L2: Business Skills          (video, research, report, code)
L3: Human Layer              (approval, notification, oversight) ← out of scope
```

## Primitives

### Core (5)

| Primitive | File/Dir | Purpose |
|---|---|---|
| **Run** | `.runs/<run_id>/` | Isolated execution directory |
| **Strategy** | `strategy.yaml` | What to execute: goal, phases, constraints |
| **Artifact** | `artifacts/*` | Single source of truth for phase outputs |
| **Constraint** | In strategy, verified at checkpoints | Typed verification: metric, semantic, script, regex, tool |
| **Event** | `journal.jsonl` | Event-sourced execution log |

### Governance (1)

| Primitive | File | Purpose |
|---|---|---|
| **Policy** | `policy.yaml` | Runtime bounds: retry, replan, timeout |

Policy answers one question: "when should the runtime stop?" It does not generate artifacts, emit events, or participate in data flow.

## Runtime Contract v1

1. Every task has a Run
2. Every Run has a Strategy
3. Every Phase consumes Artifacts (verified, not just declared)
4. Every Phase produces Artifacts
5. Every Artifact is verifiable (via Constraint)
6. Every execution emits Events (to Journal)
7. Every Runtime is bounded by Policy

## State Philosophy

**State is derived, never stored.** There is no `execution_state.json`. The current phase is derived from the journal's last `phase_completed` event. Artifact existence is checked on the filesystem. This eliminates dual-source-of-truth bugs (ADR-001).

## Recovery Protocol

When a run is interrupted (crash, restart, session timeout):

1. **Safe Anchor** — Scan journal for last `phase_completed` event
2. **Restore Counters** — Count checkpoint_failed events since anchor → `retry_count`; count strategy_invalidated across run → `replan_count`
3. **Sanitize** — Ignore artifacts from interrupted phase (they will be regenerated)
4. **Resume** — Restart from Consume Verification of target phase

No state file needed. Journal alone is sufficient (ADR-006).

## Replan Protocol

When checkpoint returns FAIL_STRATEGY (strategy is structurally wrong, not just a transient error):

1. Extract failure reason
2. Emit `strategy_invalidated` event
3. Pass `replan_context` (previous strategy + failure reason) to strategy-prompt.md
4. New strategy MUST include `[REPLAN ANALYSIS]` block: why it failed + what changed
5. Reset retry_count, resume from Phase 1 with new strategy

This prevents wasting replan attempts on structurally identical strategies (ADR-007).

## Constraint DSL

| Type | How It Verifies | Use Case |
|---|---|---|
| `metric` | Runs a tool/script, checks numeric output | "≥9 sources", "≤1500 words" |
| `semantic` | LLM reads artifact and judges | "适合抖音观众", "all claims cited" |
| `script` | Shell exit code | `check.py artifact.json` returns 0/1 |
| `regex` | Pattern match on artifact | `grep -q "## Summary" draft.md` |
| `tool` | OpenClaw tool call | Browser check, file validation |

## Architecture Decisions (ADRs)

All key decisions recorded in `references/decisions.md`:

- ADR-001: Artifact is single source of truth (no state file)
- ADR-002: Policy is governance, not core
- ADR-003: No seventh primitive
- ADR-004: Runtime doesn't own human layer
- ADR-005: Consume must be verified, not just declared
- ADR-006: Interrupt recovery from journal
- ADR-007: Replan requires failure reflection
