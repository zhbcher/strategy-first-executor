# Trajectory: Code Refactoring — Crash + Recovery

> **Task:** Refactor `payment_handler.py` (450-line monolith) into 4 modules. Ensure tests pass.
> **Run ID:** run_20260616_141500
> **Incident:** System crash during `implement` phase. Recovered via journal replay.
> **Verdict:** PASS (after recovery)

---

## strategy.yaml (first attempt — before crash)

```yaml
goal: "Refactor payment_handler.py into 4 modules: validation, gateway, logging, orchestrator. All existing tests must pass."

constraints:
  - id: tests_pass
    type: script
    condition: "pytest passes with 0 failures"
    verify: exec: cd /project && python -m pytest tests/test_payment.py -q
  - id: function_size
    type: metric
    condition: "each function ≤50 lines"
    verify: exec: python -c "
import ast, sys
with open('artifacts/refactored/__init__.py') as f:
    pass
files = ['validation.py','gateway.py','logging.py','orchestrator.py']
for fn in files:
    with open(f'artifacts/refactored/{fn}') as f:
        tree = ast.parse(f.read())
    for node in ast.walk(tree):
        if isinstance(node, ast.FunctionDef):
            end = node.end_lineno or 0
            if end - node.lineno > 50:
                print(f'{fn}:{node.name} ({end-node.lineno} lines)')
                sys.exit(1)
print('All functions ≤50 lines')
"

phases:
  - name: analyze
    actions: "Read payment_handler.py. Map all functions, classes, imports, and call dependencies. Identify extraction boundaries. Save analysis.md."
    consume: []
    output: [analysis.md]
    constraints:
      - id: coverage_complete
        type: semantic
        condition: "analysis covers all functions in payment_handler.py"
        verify: manual

  - name: plan
    actions: "Design new module structure. Assign each function to one of 4 modules. Define interfaces and import relationships. Save refactor_plan.md."
    consume: [analysis.md]
    output: [refactor_plan.md]
    constraints:
      - id: plan_complete
        type: semantic
        condition: "every function from analysis.md assigned to a module, no orphans"
        verify: manual

  - name: implement
    actions: "Write 4 module files + __init__.py. Preserve all original behavior. No new functionality."
    consume: [analysis.md, refactor_plan.md]
    output:
      - refactored/__init__.py
      - refactored/validation.py
      - refactored/gateway.py
      - refactored/logging.py
      - refactored/orchestrator.py
    constraints: [tests_pass, function_size]
```

## journal.jsonl (crash + recovery in full)

```jsonl
// === FIRST SESSION (before crash) ===
{"event":"run_started","run_id":"run_20260616_141500","timestamp":"2026-06-16T14:15:00Z"}
{"event":"phase_started","run_id":"run_20260616_141500","phase":"analyze","timestamp":"2026-06-16T14:15:05Z"}
{"event":"consume_verified","run_id":"run_20260616_141500","phase":"analyze","artifacts":[],"timestamp":"2026-06-16T14:15:05Z"}
{"event":"artifact_created","run_id":"run_20260616_141500","phase":"analyze","path":"artifacts/analysis.md","timestamp":"2026-06-16T14:18:30Z"}
{"event":"checkpoint_passed","run_id":"run_20260616_141500","phase":"analyze","constraint":"coverage_complete","evidence":"all 12 functions mapped","timestamp":"2026-06-16T14:18:32Z"}
{"event":"phase_completed","run_id":"run_20260616_141500","phase":"analyze","timestamp":"2026-06-16T14:18:32Z"}
{"event":"phase_started","run_id":"run_20260616_141500","phase":"plan","timestamp":"2026-06-16T14:18:35Z"}
{"event":"consume_verified","run_id":"run_20260616_141500","phase":"plan","artifacts":["analysis.md"],"timestamp":"2026-06-16T14:18:35Z"}
{"event":"artifact_created","run_id":"run_20260616_141500","phase":"plan","path":"artifacts/refactor_plan.md","timestamp":"2026-06-16T14:22:00Z"}
{"event":"checkpoint_passed","run_id":"run_20260616_141500","phase":"plan","constraint":"plan_complete","evidence":"12 functions assigned across 4 modules","timestamp":"2026-06-16T14:22:02Z"}
{"event":"phase_completed","run_id":"run_20260616_141500","phase":"plan","timestamp":"2026-06-16T14:22:02Z"}
{"event":"phase_started","run_id":"run_20260616_141500","phase":"implement","timestamp":"2026-06-16T14:22:05Z"}
{"event":"consume_verified","run_id":"run_20260616_141500","phase":"implement","artifacts":["analysis.md","refactor_plan.md"],"timestamp":"2026-06-16T14:22:05Z"}
{"event":"artifact_created","run_id":"run_20260616_141500","phase":"implement","path":"artifacts/refactored/__init__.py","timestamp":"2026-06-16T14:24:00Z"}
{"event":"artifact_created","run_id":"run_20260616_141500","phase":"implement","path":"artifacts/refactored/validation.py","timestamp":"2026-06-16T14:26:30Z"}
// ======== SYSTEM CRASH at 14:27:00 ========
// Gateway process killed. Partial artifacts in artifacts/refactored/:
//   __init__.py ✓
//   validation.py ✓
//   gateway.py   ✗ (not created yet)
//   logging.py   ✗ (not created yet)
//   orchestrator.py ✗ (not created yet)

// ======== SESSION RESTART at 14:35:00 ========
// Recovery protocol triggered (manifest.json exists)
// Step 1: Scan journal → last phase_completed = "plan" at 14:22:02Z
//   Anchor: after "plan" phase → Target Resume Phase: "implement"
// Step 2: Count checkpoint_failed in "implement" phase = 0 → retry_count = 0
//   Count strategy_invalidated across run = 0 → replan_count = 0
// Step 3: Sanitize — ignore artifacts/refactored/* (partial, from interrupted phase)
// Step 4: Resume from Step A (Consume Verification) of "implement" phase

// === SECOND SESSION (after recovery) ===
{"event":"phase_started","run_id":"run_20260616_141500","phase":"implement","timestamp":"2026-06-16T14:35:05Z"}
{"event":"consume_verified","run_id":"run_20260616_141500","phase":"implement","artifacts":["analysis.md","refactor_plan.md"],"timestamp":"2026-06-16T14:35:06Z"}
{"event":"artifact_created","run_id":"run_20260616_141500","phase":"implement","path":"artifacts/refactored/__init__.py","timestamp":"2026-06-16T14:35:30Z"}
{"event":"artifact_created","run_id":"run_20260616_141500","phase":"implement","path":"artifacts/refactored/validation.py","timestamp":"2026-06-16T14:37:00Z"}
{"event":"artifact_created","run_id":"run_20260616_141500","phase":"implement","path":"artifacts/refactored/gateway.py","timestamp":"2026-06-16T14:39:00Z"}
{"event":"artifact_created","run_id":"run_20260616_141500","phase":"implement","path":"artifacts/refactored/logging.py","timestamp":"2026-06-16T14:40:30Z"}
{"event":"artifact_created","run_id":"run_20260616_141500","phase":"implement","path":"artifacts/refactored/orchestrator.py","timestamp":"2026-06-16T14:42:00Z"}
{"event":"checkpoint_passed","run_id":"run_20260616_141500","phase":"implement","constraint":"tests_pass","evidence":"pytest: 34 passed, 0 failed","timestamp":"2026-06-16T14:43:00Z"}
{"event":"checkpoint_passed","run_id":"run_20260616_141500","phase":"implement","constraint":"function_size","evidence":"all functions ≤50 lines","timestamp":"2026-06-16T14:43:01Z"}
{"event":"phase_completed","run_id":"run_20260616_141500","phase":"implement","timestamp":"2026-06-16T14:43:01Z"}
{"event":"run_completed","run_id":"run_20260616_141500","timestamp":"2026-06-16T14:45:00Z"}
```

## Self-Review Output

```
## Final Self-Review

### Artifact Chain
- [✓] analysis.md — 12 functions mapped with dependencies
- [✓] refactor_plan.md — 4 modules, all functions assigned
- [✓] refactored/ — all 5 files created, 34/34 tests pass

### Constraints
- coverage_complete (semantic): PASS — all 12 functions analyzed
- plan_complete (semantic): PASS — no orphan functions
- tests_pass (script): PASS — pytest 34 passed
- function_size (metric): PASS — all ≤50 lines

### Policy
- Retries: 0
- Replans: 0
- Policy violations: none
- Incident: system crash mid-implement, recovered via journal replay

### Deviations
- Session 1 interrupted by system crash. Recovery protocol applied successfully.
  Implement phase regenerated cleanly — no partial artifacts leaked.

### Unresolved
- None

### Verdict
PASS — Recovery protocol worked as designed. Crash caused ~10 min delay but zero data loss.
```

## Key Takeaways

1. **Journal is the recovery key:** The 4-step protocol derived everything it needed from `journal.jsonl` — safe anchor, counters, phase. No separate state file required.
2. **Sanitize is critical:** Partial artifacts from the crashed session (`__init__.py`, `validation.py`) were ignored. The phase regenerated everything — no corrupted state leaked.
3. **Consume verification still works:** After recovery, `analysis.md` and `refactor_plan.md` passed consume check because they were from completed phases. The system correctly trusted them.
4. **No duplicate events:** The journal shows `artifact_created` for the same files twice (once before crash, once after) — this is fine. The runtime overwrites, not appends, artifacts.
5. **Cost of crash:** ~10 minutes of rework (the `implement` phase was re-executed from scratch). Compare to restarting the entire task, which would have been ~30 minutes.
