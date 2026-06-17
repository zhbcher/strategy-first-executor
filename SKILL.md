---
name: "strategy-first-executor"
description: "Recoverable, verifiable, resumable Task Runtime for multi-phase agentic execution on OpenClaw."
user-invocable: false
disable-model-invocation: false
metadata.openclaw:
  requires: {}
---

# Strategy-First Executor (Task Runtime)

A recoverable, verifiable, resumable Task Runtime that executes multi-phase agentic tasks on OpenClaw. Phases communicate through artifacts, not context. State is derived, never duplicated. Behavior is bounded by policy. Inspired by the StraTA framework (arxiv 2605.06642).

## Positioning

Not a Skill. Not an Execution Kernel. A **Task Runtime** — a layer between OpenClaw and business skills:

```
OpenClaw
  ↓
Task Runtime (strategy-first-executor)
  ↓
Business Skills (video, research, goal, app-builder...)
```

## Architecture Decisions

All architectural decisions are recorded in `references/decisions.md` (ADR format). Before proposing changes, read the ADRs. New proposals must answer:

- Q1: Which Primitive does this belong to?
- Q2: Does it violate any existing Contract?

## Core Primitives (5 + 1)

### Data Primitives (Core)
- **Run** — isolated execution instance (`.runs/<run_id>/`)
- **Strategy** — what to execute (`strategy.yaml`)
- **Artifact** — what was produced (files in `artifacts/`), single source of truth
- **Constraint** — typed verification conditions (`metric|semantic|script|regex|tool`)
- **Event** — what happened (`journal.jsonl`, event-sourced)

### Governance Primitive
- **Policy** — what bounds the runtime (`policy.yaml`)

## Run Directory

```
.runs/<run_id>/
├── manifest.json         # Strategy ref, artifact index, metadata
├── strategy.yaml         # Full execution strategy
├── policy.yaml           # Runtime bounds (retry, timeout, replan)
├── journal.jsonl         # Event-sourced execution journal
└── artifacts/
    ├── research.json     # Phase output
    ├── draft.md
    └── final.md
```

## Runtime Contract v1

1. Every task has a Run
2. Every Run has a Strategy
3. Every Phase consumes Artifacts (verified, not just declared)
4. Every Phase produces Artifacts
5. Every Artifact is verifiable (via Constraint)
6. Every execution emits Events (to Journal)
7. Every Runtime is bounded by Policy

## When to Apply

- Task has 3+ distinct phases with clear artifact boundaries
- Task may be interrupted and needs resumability
- Multiple similar tasks may run concurrently (Run ID isolation)
- Previous attempts showed context-based drift

## Workflow

### Phase 0: Run Setup

1. Generate `run_id` (timestamp-based)
2. Create `.runs/<run_id>/` directory structure
3. If `.runs/<run_id>/manifest.json` exists → resume from last incomplete phase
4. Otherwise → proceed to Strategy Generation

### Phase 1: Strategy + Policy Generation

1. Generate strategy using `references/strategy-prompt.md`, save as `strategy.yaml`
2. Write `policy.yaml` with default bounds (see `references/policy.md`)
3. Write `manifest.json`
4. Append `run_started` event to journal

### Phase 2: Execution Loop

For each phase:

**Step A: Consume Verification (MANDATORY)**
1. Read `consume` list from strategy for this phase
2. For each file: verify it exists in manifest AND on filesystem
3. Any missing → emit `consume_validation_failed` event, STOP
4. All present → emit `consume_verified` event, continue

**Step B: Execute**
1. Execute phase actions using declared tools
2. Write all `output` artifacts
3. Emit `artifact_created` event for each output

**Step C: Constraint Verification**
1. Run checkpoint gate via `references/checkpoint-gate-prompt.md`
2. Gate tracks retry/replan counts against `policy.yaml`
3. On FAIL: fix, increment retry_count, re-run gate
4. On FAIL_STRATEGY: increment replan_count, regenerate strategy
5. Policy violation → emit `policy_violation`, STOP
6. All PASS → emit `phase_completed`, proceed

### Phase 3: Final Self-Review

1. Run `references/self-review-prompt.md`
2. Verify artifact chain integrity
3. Emit `run_completed` event

## File Specifications

### policy.yaml

```yaml
max_retry: 3
max_replan: 2
phase_timeout: 30m
max_total_runtime: 120m
```

Policy is the Governance Primitive. It does not generate artifacts or events. Its sole responsibility is to stop the runtime. Never add approval/notification/UI fields here.

### manifest.json

```json
{
  "run_id": "run_20260617_120000",
  "strategy": "strategy.yaml",
  "policy": "policy.yaml",
  "artifacts": ["research.json", "draft.md", "final.md"],
  "journal": "journal.jsonl",
  "created_at": "...",
  "updated_at": "..."
}
```

manifest is an index, not a state store. Current phase is derived from journal events.

## Constraint DSL

| Type | Verification | Example |
|------|-------------|---------|
| `metric` | Tool/script | `source_count >= 9` |
| `semantic` | LLM judgment | "适合抖音观众" |
| `script` | Shell exit code | `exec: check.py artifact.json` |
| `regex` | Pattern match | `grep -q "## Summary" draft.md` |
| `tool` | OpenClaw tool | `browser check page title` |

## References

- `references/decisions.md` — architecture decision records (ADRs)
- `references/strategy-prompt.md` — strategy generation prompt
- `references/checkpoint-gate-prompt.md` — typed constraint verification with policy enforcement
- `references/self-review-prompt.md` — final review against manifest + journal + artifacts
- `references/policy.md` — policy specification and enforcement rules
- StraTA paper: arxiv.org/abs/2605.06642
