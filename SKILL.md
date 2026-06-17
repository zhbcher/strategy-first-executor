---
name: "strategy-first-executor"
description: "Wrap complex multi-step tasks with explicit strategy-first execution, checkpoint gating, and self-review."
user-invocable: false
disable-model-invocation: false
metadata.openclaw:
  requires: {}
---

# Strategy-First Executor

Reliable multi-phase task execution with explicit strategy, state persistence, artifact chains, checkpoint gating, and structured journaling. Inspired by the StraTA framework (arxiv 2605.06642).

## When to Apply

- Task has 3+ distinct phases (research → analyze → output)
- Task spans multiple sub-agent spawns that need coordination
- Task may be interrupted and needs resumability
- Previous attempts showed drift, inconsistency, or forgetting

Skip for single-step or trivial tasks.

## Architecture

```
Task
 ↓
Resume Check (execution_state.json)
 ↓
Strategy Generator (Phase 0)
 ↓  writes execution_state.json
Execution (Phase 1→N)
 ├─ Read input artifact(s)
 ├─ Execute actions
 ├─ Write output artifact
 ├─ Checkpoint Gate (tool verify → PASS/FAIL/FAIL_STRATEGY)
 ├─ Update execution_state.json
 └─ Append execution_journal.jsonl
 ↓
Final Self-Review (Phase N+1)
```

## Key Design Decisions

### State Persistence

Every phase writes `execution_state.json` to a fixed path. Resume from this file on restart.

### Artifact Chain

Phases do NOT communicate through context. Each phase declares inputs and outputs:

```
Phase 1 → research.json
Phase 2 → reads research.json → writes draft.md
Phase 3 → reads draft.md → writes final.md
```

### Structured Checkpoints

Numeric/machine-checkable checkpoints use structured format. Semantic checkpoints use natural language.

### Execution Journal

Every action appends to `execution_journal.jsonl`. Recoverable, debuggable, auditable.

## Workflow

### Phase 0: Resume Check

1. Check if `<task_path>/execution_state.json` exists
2. If yes, read current state, skip completed phases, resume from first `running` or `failed` phase
3. If no, proceed to Strategy Generation

### Phase 1: Strategy Generation

Generate and save strategy using `references/strategy-prompt.md`. Write `execution_state.json` with all phases `pending`.

### Phase 2: Execution Loop

For each phase:

1. Read input artifacts from previous phases
2. Execute actions
3. **Write output artifact** to declared path
4. Run checkpoint gate via `references/checkpoint-gate-prompt.md`
5. If `Verify: <tool_call>` is specified, run it before judging
6. Update `execution_state.json`
7. Append to `execution_journal.jsonl`
8. FAIL → fix and re-run gate. FAIL_STRATEGY → regenerate from Phase 1

### Phase 3: Final Self-Review

After all phases, run `references/self-review-prompt.md`. Review against journal and artifacts.

## File Convention

All paths relative to `<task_path>/`:

```
task_path/
├── execution_state.json      # Phase status + artifact refs
├── execution_journal.jsonl   # Append-only action log
├── <phase_1_output>          # Declared in strategy
├── <phase_2_output>
└── ...
```

### execution_state.json

```json
{
  "task": "task description",
  "strategy": "full strategy text",
  "phases": {
    "research": {"status": "passed", "artifact": "research.json", "updated_at": "..."},
    "draft": {"status": "running", "artifact": null, "updated_at": "..."},
    "polish": {"status": "pending", "artifact": null, "updated_at": null}
  },
  "created_at": "...",
  "updated_at": "..."
}
```

Status values: `pending | running | passed | failed | skipped`

### execution_journal.jsonl

Append-only, one line per action:

```jsonl
{"phase":"research","action":"web_search","result":"success","evidence":"9 results","timestamp":"..."}
{"phase":"research","action":"checkpoint_gate","result":"pass","evidence":"source_count=9","timestamp":"..."}
```

## Strategy Format

```
## Strategy

### Goal
[One sentence: definition of done]

### Constraints
- [Verifiable condition] → Verify: [tool call, script, or "manual"]

### Execution Plan

#### Phase: <name>
Actions: [what to do, tools to use]
Input: <previous_phase_output or null>
Output: <this_phase_output_path>
🔒 Checkpoint:
  condition: [natural language for semantic, structured for numeric]
  verify: [tool call if applicable, or "manual"]

#### Phase: <name>
...
```

**Structured checkpoint format** (for numeric/machine-checkable conditions):

```yaml
🔒 Checkpoint:
  metric: <metric_name>
  operator: ">=" | "<=" | "==" | "!=" | ">"
  value: <number>
  verify: exec: <script_path> <artifact_path>
```

## Sub-agent Integration

When spawning sub-agents for parallel work:
- Inject the strategy + execution_state.json into each sub-agent
- Each sub-agent writes its own artifact, updates a shared journal
- Orchestrator aggregates artifacts and runs final review

## References

- `references/strategy-prompt.md` — strategy generation with artifact declarations
- `references/checkpoint-gate-prompt.md` — checkpoint verification with tool support
- `references/self-review-prompt.md` — final review against journal + artifacts
- StraTA paper: arxiv.org/abs/2605.06642
