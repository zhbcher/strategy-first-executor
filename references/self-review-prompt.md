# Self-Review Prompt

Use this prompt after ALL phases complete. Review against strategy, artifacts, and event journal.

---

**Run ID:** {run_id}
**Strategy:** {strategy_path}
**Manifest:** {manifest_path}
**Journal:** {journal_path}
**Artifacts:** {artifact_list}

## Review

### 1. Artifact Chain Integrity

Verify the consume/output chain. For each phase:
- Every `consume` artifact exists in `artifacts/`
- Every `output` artifact was created
- No broken links in the chain

### 2. Constraint Verification Summary

From the journal, list every constraint and its final status:

```
- {constraint_id} ({type}): {passed/failed} — {evidence from journal}
```

### 3. Deviations

Any action that neither followed the strategy nor advanced the task. Justified deviations (new information invalidated strategy) are OK.

### 4. Verdict

```
## Final Self-Review

### Artifact Chain
- [chain_status]

### Constraints
- [constraint_summary]

### Deviations
- [list or "none"]

### Unresolved
- [list or "none"]

### Verdict
[PASS / NEEDS_FIX — explanation]
```
