# Checkpoint Gate Prompt

Use this prompt at every checkpoint between execution phases. Do not proceed past a FAIL.

---

You have completed Phase {phase_number}: {phase_name}.

The strategy requires this checkpoint to pass before proceeding:

**Checkpoint:** {checkpoint_condition}

**What was produced in this phase:** {phase_output}

Verify the checkpoint condition against what was actually produced. Respond with ONLY:

- `PASS` — the condition is met, proceed to next phase
- `FAIL: <reason>` — the condition is not met, state what's missing/wrong
- `FAIL_STRATEGY: <reason>` — the strategy itself is invalid given new information

Do not discuss. Do not suggest fixes. Answer with PASS, FAIL, or FAIL_STRATEGY only.
