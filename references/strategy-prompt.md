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

Output ONLY the strategy in this format:

```
## Strategy

### Goal
[One sentence: what success looks like]

### Constraints
- [Hard rules]
- [Format requirements]
- [Things to avoid]

### Execution Plan
1. [Phase name] → [specific actions, tools, expected outcome]
2. [Phase name] → [specific actions, tools, expected outcome]
...

### Checkpoints
- After step N: verify [condition]
- After step M: verify [condition]
```

Do not execute any actions yet. Output only the strategy.
