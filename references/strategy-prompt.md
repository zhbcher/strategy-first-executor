# Strategy Generation Prompt

Use this prompt at the start of every execution. Output strategy.yaml that the runtime will follow.

---

You are about to execute a complex multi-phase task. Before any action, generate a complete execution strategy.

**Task:** {task_description}

**Relevant Context:** {context}

**Run ID:** {run_id}

**Work directory:** .runs/{run_id}/

Your strategy must declare:
1. **Goal** — one sentence: what success looks like
2. **Constraints** — typed, verifiable. Each has: id, type (metric|semantic|script|regex|tool), condition, verify method
3. **Phases** — sequential. Each has: name, actions, consume (input artifacts), output (output artifacts), constraints

Output ONLY the strategy in this YAML format:

```yaml
goal: "One sentence definition of done"

constraints:
  - id: <constraint_id>
    type: metric|semantic|script|regex|tool
    condition: <description>
    verify: <tool_call or "manual">

phases:
  - name: <phase_name>
    actions: "<specific actions, tools to use>"
    consume: []
    output: [<output_file>]
    constraints: [<constraint_id>, ...]

  - name: <phase_name>
    actions: "<specific actions>"
    consume: [<previous_output_file>]
    output: [<output_file>]
    constraints: [<constraint_id>, ...]
```

Rules:
- First phase has `consume: []`
- Every subsequent phase consumes at least one artifact from prior phases
- Numeric checks use `metric` type with script verification
- Format/quality checks use `semantic` type
- File pattern checks use `regex` type

After outputting the strategy:
1. Save as `.runs/{run_id}/strategy.yaml`
2. Write `policy.yaml` with defaults: max_retry=3, max_replan=2, phase_timeout=30m, max_total_runtime=120m
3. Create `manifest.json` with run metadata
4. Create `journal.jsonl` with `run_started` event
5. Create `artifacts/` directory

Do not execute any phase actions yet.
