# Checkpoint Gate Prompt

Use this prompt at every checkpoint between phases. Never proceed past a FAIL.

---

**Run ID:** {run_id}
**Phase:** {phase_name}
**Output artifacts:** {artifact_list}
**Next phase consume:** {next_consume_list}

## Constraints to Verify

{constraints_block}

---

## Protocol

### 1. For each constraint

| Constraint Type | Action |
|----------------|--------|
| `metric` | Run the verify tool NOW. Parse output. Judge based on tool result, not opinion. |
| `script` | Run the verify script NOW. Judge based on exit code + stdout. |
| `regex` | Run the verify command NOW. Judge based on match/no-match. |
| `tool` | Execute the OpenClaw tool call NOW. Judge based on tool output. |
| `semantic` | Read the output artifact. Judge based on the condition vs artifact contents. |

### 2. For each constraint, respond with ONE line

```
CONSTRAINT {constraint_id}: PASS
```
```
CONSTRAINT {constraint_id}: FAIL — <specific reason, what is missing/wrong>
```

### 3. After ALL constraints are evaluated

If ALL PASS:
- Write `phase_completed` event to `.runs/{run_id}/journal.jsonl`
- Proceed to next phase

If ANY FAIL:
- Write `checkpoint_failed` event with failing constraint IDs
- Fix the issues
- Re-run this gate from scratch

If FAIL_STRATEGY (strategy itself is wrong):
- Write `strategy_invalidated` event
- Regenerate strategy from scratch

### 4. Journal events to emit

```jsonl
{"event":"checkpoint_passed","run_id":"{run_id}","phase":"{phase_name}","constraint":"{constraint_id}","evidence":"<tool output or reason>","timestamp":"ISO8601"}
{"event":"checkpoint_failed","run_id":"{run_id}","phase":"{phase_name}","constraint":"{constraint_id}","reason":"<reason>","timestamp":"ISO8601"}
{"event":"phase_completed","run_id":"{run_id}","phase":"{phase_name}","timestamp":"ISO8601"}
{"event":"strategy_invalidated","run_id":"{run_id}","phase":"{phase_name}","reason":"<reason>","timestamp":"ISO8601"}
```

Do not discuss. Execute the protocol.
