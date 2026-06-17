# Runtime Policy

Policy is the Governance Primitive. It bounds the runtime. It does not participate in the data flow.

## Format (policy.yaml)

```yaml
max_retry: 3           # Max checkpoint FAIL retries per phase
max_replan: 2          # Max FAIL_STRATEGY replans per run
phase_timeout: 30m     # Max wall-clock time per phase
max_total_runtime: 120m  # Max total run wall-clock time
```

## Rule: Only Stopping Conditions

Policy fields must answer: "when should the runtime stop?"

Never add to Policy:
- Approval rules → Human Layer (L3)
- Notification settings → Human Layer (L3)
- UI configuration → Human Layer (L3)
- Memory settings → Platform (L0)

## Enforcement

### Retry Guard (per-phase)

```
Checkpoint FAIL → retry_count++
  if retry_count > max_retry:
    emit policy_violation event
    STOP
  else:
    fix and re-run gate
```

### Replan Guard (per-run)

```
FAIL_STRATEGY → replan_count++
  if replan_count > max_replan:
    emit policy_violation event
    STOP
  else:
    regenerate strategy
    reset retry_count for new execution
```

### Timeout Guard

```
Phase elapsed > phase_timeout → emit policy_violation, STOP
Run elapsed > max_total_runtime → emit policy_violation, STOP
```

## Default Values

If policy.yaml is not found:

```yaml
max_retry: 3
max_replan: 2
phase_timeout: 30m
max_total_runtime: 120m
```
