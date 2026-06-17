# Checkpoint Gate Prompt

Use this prompt at every checkpoint between phases. Never proceed past a FAIL.

---

You have completed Phase: **{phase_name}**.

**Checkpoint condition:** {checkpoint_condition}

**Verification method:** {verification_method}

**Output artifact:** {artifact_path}

**Expected input for next phase:** {next_phase_input}

## Protocol

1. **If verification method is a tool call** (e.g., `exec: python check.py artifact.json`):
   - Run the tool NOW
   - Capture its output as evidence
   - Judge PASS/FAIL based on the tool output, not your opinion

2. **If verification is "manual"**:
   - Read the output artifact
   - Judge PASS/FAIL based on the checkpoint condition vs artifact contents

3. **Respond with ONE line only**:

```
PASS
```

```
FAIL: <specific reason, what is missing/wrong>
```

```
FAIL_STRATEGY: <why the strategy itself is no longer valid>
```

4. **After PASS**:
   - Mark phase as `passed` in `execution_state.json`
   - Set next phase to `running`
   - Append to `execution_journal.jsonl`: `{"phase":"{phase_name}","action":"checkpoint_gate","result":"pass","evidence":"<evidence>","timestamp":"<now>"}`
   - Proceed to next phase

5. **After FAIL**:
   - Fix the issue
   - Re-run this gate
   - Do NOT mark the phase as passed until the gate passes

6. **After FAIL_STRATEGY**:
   - Regenerate strategy from scratch
   - Reset all phase states

Do not discuss. Do not suggest fixes outside the fix loop. Execute the protocol.
