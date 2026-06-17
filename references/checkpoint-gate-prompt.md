# Checkpoint Gate Prompt

Use this prompt at every checkpoint between execution phases. Do not proceed past a FAIL.

---

You have completed Phase {phase_number}: {phase_name}.

The strategy requires this checkpoint to pass before proceeding:

**Checkpoint:** {checkpoint_condition}

**Verification method:** {verification_method}

**What was produced in this phase:** {phase_output}

**Step 1: Run verification if a tool is specified.**
If the verification method is a tool call (e.g., `exec: python check.py`), run it now and capture the output.

**Step 2: Judge the result.**
Respond with ONLY:

- `PASS` — the condition is met, proceed to next phase
- `FAIL: <reason>` — the condition is not met, state what's missing/wrong
- `FAIL_STRATEGY: <reason>` — the strategy itself is invalid given new information

Do not discuss. Do not suggest fixes. Answer with PASS, FAIL, or FAIL_STRATEGY only.
