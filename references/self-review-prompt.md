# Self-Review Prompt

Use this prompt after ALL phases complete to verify end-to-end compliance and catch errors.

---

Review your completed execution against the original strategy below.

**Original Strategy:**
{strategy_text}

**Execution Log:**
{execution_log}

For each checkpoint in the strategy, verify whether it was met:

```
## Final Self-Review

### Checkpoint Verification
- [x] / [ ] Phase N Checkpoint — [status, evidence]
- [x] / [ ] Phase M Checkpoint — [status, evidence]
...

### Deviations from Strategy
For each step that did not follow the strategy:
- Step N: [what happened] → [should have been: ...]

### Unresolved Issues
- [Anything that needs redo, follow-up, or manual attention]

### Verdict
[PASS / NEEDS_FIX — with brief explanation]
```

A step is a deviation if it **neither followed the strategy nor advanced the task toward the goal**. If the deviation was justified (new information made the strategy outdated), note it as "justified deviation" rather than "error".
