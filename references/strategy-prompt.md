# Strategy Generation Prompt

Use this prompt at the start of any strategy-first execution. The strategy will be saved as the execution plan and persisted across phases.

---

You are about to execute a complex multi-step task. Before you begin, generate a comprehensive execution strategy.

**Task:** {task_description}

**Relevant Context:** {context}

**Work directory:** {task_path}

Your strategy must declare:
1. Concrete constraints — each verifiable (yes/no condition)
2. Phase-by-phase execution plan — each phase declares its input artifact, output artifact, and checkpoint
3. Checkpoints — gated, each phase must pass before the next begins

For numeric/machine-checkable conditions, use structured checkpoint format. For semantic conditions, use natural language.

Output ONLY the strategy in this format:

```
## Strategy

### Goal
[One sentence: what success looks like]

### Constraints
- [Verifiable condition] → Verify: [tool call or "manual"]
- [Verifiable condition] → Verify: [tool call or "manual"]

### Execution Plan

#### Phase: <name>
Actions: [specific actions, tools]
Input: <file_path or null>
Output: <file_path>
🔒 Checkpoint:
  condition: [natural language]
  verify: ["manual" or tool call]

#### Phase: <name>
Actions: [specific actions, tools]
Input: <previous_output_file>
Output: <file_path>
🔒 Checkpoint:
  metric: <name>
  operator: ">="
  value: <number>
  verify: exec: <script> <file_path>
...
```

After outputting the strategy, save it and create `execution_state.json` at `{task_path}` with all phases set to `pending`.

Do not execute any actions yet.
