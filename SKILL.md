---
name: "strategy-first-executor"
description: "Recoverable, verifiable, resumable Task Runtime for multi-phase agentic execution on OpenClaw."
user-invocable: false
disable-model-invocation: false
metadata.openclaw:
  requires: {}
---

# Strategy-First Executor (Task Runtime)

A recoverable, verifiable, resumable Task Runtime that executes multi-phase agentic tasks on OpenClaw. Phases communicate through artifacts, not context. State is derived, never duplicated. Inspired by the StraTA framework (arxiv 2605.06642).

## Positioning

Not a Skill. Not an Execution Kernel. A **Task Runtime** — a layer between OpenClaw and business skills:

```
OpenClaw
  ↓
Task Runtime (strategy-first-executor)
  ↓
Business Skills (video, research, goal, app-builder...)
```

## Run Model

Every execution is an isolated **Run**:

```
.runs/<run_id>/
├── manifest.json         # Strategy ref, artifact index, metadata
├── strategy.yaml         # Full execution strategy
├── journal.jsonl         # Event-sourced execution journal
└── artifacts/
    ├── research.json     # Phase 1 output
    ├── draft.md          # Phase 2 output
    └── final.md          # Phase 3 output
```

## When to Apply

- Task has 3+ distinct phases with clear artifact boundaries
- Task may be interrupted and needs resumability
- Multiple similar tasks may run concurrently (Run ID isolation)
- Previous attempts showed context-based drift

## Key Design Decisions

### 1. Artifact as Single Source of Truth

Phase state is **never stored**. It is **derived** from artifact existence:

```python
# Never:
state = {"phase_1": "done"}

# Always:
state = derived_from(artifacts_exist, journal_events)
```

`manifest.json` is an index, not a state store. No dual-write conflicts.

### 2. Constraint DSL

Checkpoints use typed constraints, not raw natural language:

| Type | Verification | Example |
|------|-------------|---------|
| `metric` | Tool/script | `source_count >= 9` |
| `semantic` | LLM judgment | "适合抖音观众" |
| `script` | Shell exit code | `exec: check.py artifact.json` |
| `regex` | Pattern match | `grep -q "## Summary" draft.md` |
| `tool` | OpenClaw tool call | `browser check page title` |

### 3. Event-Sourced Journal

Every state change emits an event. State is replayable:

```jsonl
{"event":"run_started","run_id":"...","timestamp":"..."}
{"event":"phase_started","phase":"research","timestamp":"..."}
{"event":"artifact_created","phase":"research","path":"research.json","timestamp":"..."}
{"event":"checkpoint_passed","phase":"research","constraint":"source_count","timestamp":"..."}
{"event":"phase_completed","phase":"research","timestamp":"..."}
```

### 4. DAG-Ready Artifacts

Each phase declares `consume` (what it reads) and `output` (what it writes). This is a DAG edge:

```yaml
phase_2:
  consume: [research.json]
  output: [draft.md]
```

Current implementation is sequential. `consume` enables future parallel phase execution.

## Workflow

### Phase 0: Run Setup

1. Generate `run_id` (timestamp-based, e.g., `run_20260617_120000`)
2. Create `.runs/<run_id>/` directory structure
3. If `.runs/<run_id>/manifest.json` exists → resume (skip to last incomplete phase)
4. Otherwise → proceed to Strategy Generation

### Phase 1: Strategy Generation

1. Generate strategy using `references/strategy-prompt.md`
2. Save as `strategy.yaml`
3. Write `manifest.json` with run metadata, artifact list, constraint registry
4. Append `run_started` event to journal

### Phase 2: Execution Loop

For each phase (sequential, respecting consume/output DAG):

1. Read declared `consume` artifacts — verify they exist
2. Execute phase actions
3. Write `output` artifact(s)
4. Emit `artifact_created` event(s) to journal
5. Run all declared constraints via `references/checkpoint-gate-prompt.md`
6. For `metric|script|regex` constraints: run tool BEFORE model judgment
7. For `semantic` constraints: model judgment with artifact evidence
8. Emit `checkpoint_passed` or `checkpoint_failed` event
9. On FAIL: fix and re-run. On FAIL_STRATEGY: regenerate strategy.
10. Emit `phase_completed` event

### Phase 3: Final Self-Review

1. Run `references/self-review-prompt.md`
2. Review against strategy + manifest + journal
3. Verify artifact chain integrity (every consume has a matching output from prior phase)
4. Emit `run_completed` event

## Constraint Format (in strategy.yaml)

```yaml
constraints:
  - id: citation_check
    type: metric
    condition: uncited_claims == 0
    verify: exec: python check_citations.py draft.md

  - id: word_limit
    type: metric
    condition: word_count <= 1500
    verify: exec: wc -w draft.md

  - id: audience_tone
    type: semantic
    condition: "适合抖音观众，口语化，有网感"
    verify: manual

  - id: has_summary
    type: regex
    condition: "document contains ## Summary section"
    verify: exec: grep -q '## Summary' final.md
```

## Strategy Format (strategy.yaml)

```yaml
goal: "One sentence: definition of done"

constraints:
  - id: <constraint_id>
    type: metric|semantic|script|regex|tool
    condition: <condition_text>
    verify: <tool_call or "manual">

phases:
  - name: research
    actions: "web_search for each section topic, collect 3+ sources per section"
    consume: []
    output: [research.json]
    constraints: [source_count]

  - name: draft
    actions: "write draft.md with citations from research.json"
    consume: [research.json]
    output: [draft.md]
    constraints: [citation_check, word_limit]

  - name: polish
    actions: "fix formatting, check URLs, final read-through"
    consume: [draft.md]
    output: [final.md]
    constraints: [has_summary, audience_tone]
```

## manifest.json

```json
{
  "run_id": "run_20260617_120000",
  "strategy": "strategy.yaml",
  "artifacts": ["research.json", "draft.md", "final.md"],
  "journal": "journal.jsonl",
  "created_at": "2026-06-17T12:00:00Z",
  "updated_at": "2026-06-17T12:00:00Z",
  "current_phase": null
}
```

`current_phase` is the last phase that emitted a `phase_completed` event. The next phase to execute is `current_phase + 1`. If null, execution has not started.

## References

- `references/strategy-prompt.md` — strategy generation with artifact + constraint declarations
- `references/checkpoint-gate-prompt.md` — typed constraint verification with tool support
- `references/self-review-prompt.md` — final review against manifest + journal + artifacts
- StraTA paper: arxiv.org/abs/2605.06642
