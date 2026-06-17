# Self-Review Prompt

Use this prompt after ALL phases complete.

---

**Run ID:** {run_id}
**Strategy:** {strategy_path}
**Policy:** {policy_path}
**Manifest:** {manifest_path}
**Journal:** {journal_path}
**Artifacts:** {artifact_list}

## Review

### 1. Artifact Chain Integrity

For each phase, verify: every `consume` artifact exists, every `output` artifact created.

### 2. Constraint Summary

From journal, list each constraint and final status:

```
- {constraint_id} ({type}): {passed/failed} — {evidence}
```

### 3. Policy Compliance

Check for policy_violation events:

```
- Policy violations: [none / list]
```

### 4. Deviations

Actions that neither followed strategy nor advanced task.

### 5. Verdict

```
## Final Self-Review

### Artifact Chain
- [status]

### Constraints
- [summary]

### Policy
- [status]

### Deviations
- [list or "none"]

### Unresolved
- [list or "none"]

### Verdict
[PASS / NEEDS_FIX]
```
