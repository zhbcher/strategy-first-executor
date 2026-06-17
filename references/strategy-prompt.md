# Strategy Generation Prompt

Use this prompt at the start of every execution. Output strategy.yaml that the runtime will follow.

---

You are about to execute a complex multi-phase task. Before any action, generate a complete execution strategy.

**Task:** {task_description}

**Relevant Context:** {context}

**Run ID:** {run_id}

**Work directory:** .runs/{run_id}/

<!-- REPLAN_CONTEXT_START — only present when replanning -->
{replan_context}
<!-- REPLAN_CONTEXT_END -->

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

---

## REPLANNING MODE

If `replan_context` is present (previous strategy failed), you MUST:

1. **Analyze the failure**: Read the previous strategy and failure reason. Write 1-2 sentences on WHY it failed.
2. **State your fix**: Explicitly say what you changed to avoid that failure path.

Output this analysis as a comment block BEFORE the strategy:

```yaml
# [REPLAN ANALYSIS]
# Why it failed: <your analysis>
# What changed: <your concrete fix>
---
# <strategy.yaml follows>
```

Do NOT output a strategy that is structurally identical to the failed one. If the failure reason suggests the task itself is impossible, say so explicitly in the analysis instead of generating another doomed strategy.

---

After outputting the strategy:
1. Save as `.runs/{run_id}/strategy.yaml`
2. If this is not a replan: write `policy.yaml` with defaults
3. Create/update `manifest.json` with run metadata
4. Append `run_started` or `strategy_invalidated` event to journal
5. Create `artifacts/` directory if not exists

Do not execute any phase actions yet.
