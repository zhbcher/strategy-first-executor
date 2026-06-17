# Strategy Generation Prompt

Use this prompt at the start of any strategy-first execution.

---

You are about to execute a complex multi-step task. Before you begin, generate a comprehensive strategy that will guide every subsequent action.

**Task:** {task_description}

**Relevant Context:** {context}

Your strategy must be:
1. **Concrete** — future actions can be checked against it step by step
2. **Practical** — based only on known information, no guessing future data
3. **Complete** — covers the full task from start to finish

**Constraints must be verifiable.** Each constraint must be a yes/no condition that can be checked objectively. Bad: "be careful with X". Good: "output value must equal reference value, deviation >0.1s = error".

**Checkpoints must be gated.** Every key phase transition must have a checkpoint. The agent will not proceed past a checkpoint until it passes.

Output ONLY the strategy in this format:

```
## Strategy

### Goal
[One sentence: what success looks like]

### Constraints
- [Verifiable yes/no condition]
- [Verifiable yes/no condition]

### Execution Plan
1. [Phase name] → [actions, tools, expected output] 🔒 Checkpoint: [verifiable condition]
2. [Phase name] → [actions, tools, expected output] 🔒 Checkpoint: [verifiable condition]
...

### Checkpoints
- After Phase N: verify [specific measurable condition]
- After Phase M: verify [specific measurable condition]
```

Do not execute any actions yet. Output only the strategy.
