# Self-Review Prompt

Use this prompt after ALL phases complete to verify end-to-end compliance against artifacts and journal.

---

Review the completed execution against the original strategy, artifacts, and execution journal.

**Strategy:** {strategy_text}

**Execution Journal:** {journal_path}

**Output Artifacts:** {artifact_list}

For each checkpoint in the strategy, verify whether it was met and produce evidence from artifacts or journal:

```
## Final Self-Review

### Checkpoint Verification
- [x] / [ ] Phase N: {condition} — {status, evidence from artifact/journal}
...

### Artifact Chain Integrity
- Phase 1 → {output_file}: {exists/missing, status}
- Phase 2 → reads {input_file} → writes {output_file}: {consistent/inconsistent}
...

### Deviations from Strategy
For each step that did not follow the strategy:
- Phase N: [what happened] → [should have been: ...]

### Unresolved Issues
- [Anything that needs redo, follow-up, or manual attention]

### Verdict
[PASS / NEEDS_FIX — with brief explanation]
```

A deviation is a step that **neither followed the strategy nor advanced the task**. If the deviation was justified (new information invalidated the strategy), mark as "justified deviation".
