# Checkpoint Gate Prompt

Use this prompt at every checkpoint. Never proceed past a FAIL. Policy violations are terminal.

---

**Run ID:** {run_id}
**Phase:** {phase_name}
**Output artifacts:** {artifact_list}
**Next phase consume:** {next_consume_list}

**Policy:** max_retry={max_retry} (current={retry_count}), max_replan={max_replan} (current={replan_count}), phase_timeout={phase_timeout}

## Constraints

{constraints_block}

---

## Protocol

### Step 0: Policy Check (BEFORE verification)

```
If retry_count >= max_retry:
  Write: {"event":"policy_violation","run_id":"{run_id}","phase":"{phase_name}","violation":"max_retry","limit":{max_retry},"actual":{retry_count},"timestamp":"<now>"}
  Respond: POLICY_VIOLATION: max_retry ({max_retry}) exceeded. Run terminated.
  STOP

If replan_count >= max_replan:
  Write: {"event":"policy_violation","run_id":"{run_id}","phase":"{phase_name}","violation":"max_replan","limit":{max_replan},"actual":{replan_count},"timestamp":"<now>"}
  Respond: POLICY_VIOLATION: max_replan ({max_replan}) exceeded. Run terminated.
  STOP
```

### Step 1: Consume Verification

```
For each file in next_consume_list:
  Check: declared in manifest? + exists on filesystem?
  
  If ANY missing:
    Write: {"event":"consume_validation_failed","run_id":"{run_id}","phase":"<next_phase>","missing":["<files>"],"timestamp":"<now>"}
    Respond: CONSUME_VALIDATION_FAILED: missing [<files>]. Run terminated.
    STOP

  All present:
    Write: {"event":"consume_verified","run_id":"{run_id}","phase":"<next_phase>","artifacts":["<files>"],"timestamp":"<now>"}
```

### Step 2: Constraint Verification

| Type | Action |
|------|--------|
| `metric` | Run the verify tool. Judge based on tool output. |
| `script` | Run the verify script. Judge based on exit code + stdout. |
| `regex` | Run the verify command. Judge based on match/no-match. |
| `tool` | Execute the OpenClaw tool call. Judge based on tool output. |
| `semantic` | Read the output artifact. Judge based on condition vs contents. |

Respond per constraint:

```
CONSTRAINT {id}: PASS
CONSTRAINT {id}: FAIL — <reason>
```

### Step 3: Outcome

**ALL PASS:**
```
Write checkpoint_passed events
Write phase_completed event
Proceed to next phase
```

**ANY FAIL:**
```
retry_count++
Write checkpoint_failed event(s)
Fix issues. Re-run from Step 0.
```

**FAIL_STRATEGY:**
```
replan_count++
Write strategy_invalidated event
Regenerate strategy from Phase 1
```

Do not discuss. Execute the protocol.
