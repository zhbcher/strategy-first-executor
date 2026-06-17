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
2. **Constraints** — typed, verifiable. Each constraint has: id, type (metric|semantic|script|regex|tool), condition, verify method
3. **Phases** — sequential. Each phase has: name, actions, consume (input artifacts), output (output artifacts), constraints (which constraint IDs to check)

Output ONLY the strategy in this YAML format:

```yaml
goal: "One sentence definition of done"

constraints:
  - id: <constraint_id>
    type: metric|semantic|script|regex|tool
    condition: <description>
    verify: <tool_call or "manual">

  - id: <constraint_id>
    type: semantic
    condition: <description>
    verify: manual

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
- External validation uses `tool` type

After outputting the strategy:
1. Save as `.runs/{run_id}/strategy.yaml`
2. Create `.runs/{run_id}/manifest.json` with run metadata
3. Create `.runs/{run_id}/journal.jsonl` with `run_started` event
4. Create `.runs/{run_id}/artifacts/` directory

Do not execute any phase actions yet.
