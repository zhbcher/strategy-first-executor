# Contributing to Strategy-First Executor

Thanks for your interest. This project grows through shared strategy templates, real-world execution trajectories, and architectural improvements.

## Ways to Contribute

### 1. Share a Strategy Template

Used the runtime for a task domain not yet covered? Add it to `examples/strategies/`.

**Format:**
```markdown
# Strategy: {domain name}

## When to Use
{1-2 sentences describing the task pattern}

## Typical Policy
```yaml
max_retry: {n}
max_replan: {n}
phase_timeout: {duration}
```

## Strategy Template
```yaml
goal: "..."

constraints:
  - id: ...
    type: metric|semantic|script|regex|tool
    condition: ...
    verify: ...

phases:
  - name: ...
    actions: "..."
    consume: []
    output: [...]
    constraints: [...]
```

## Notes
{Any domain-specific tips, common failure modes, replan strategies}
```

Place in: `examples/strategies/{domain}.md`

### 2. Share an Execution Trajectory

Ran a task end-to-end? Submit the full journal + artifacts as a trajectory example.

**What to include:**
- `strategy.yaml` (the strategy used)
- `policy.yaml`
- `journal.jsonl` (anonymized, but keeping event structure)
- Key artifacts (or summaries if too large)
- Self-review output
- Notes on what went well / what was surprising

Place in: `examples/trajectories/{run-id}-description.md`

### 3. Report Issues

Found a bug or have a feature idea? Tell us:
- What you were trying to do
- What strategy you used
- What happened vs. what you expected
- If a recovery/replan was involved, include the journal snippet

### 4. Propose Architecture Changes

Read `references/decisions.md` first. Proposals must answer:
- **Q1:** Which Primitive does this belong to? (Run, Strategy, Artifact, Constraint, Event, Policy)
- **Q2:** Does it violate any existing Contract?

ADR-003: No seventh primitive. New capabilities must attach to existing ones.

## Style Guide

- Strategy files: YAML with comments, 2-space indent
- Journal events: One JSON object per line, ISO 8601 timestamps
- Documentation: Markdown, Chinese or English OK
- Keep constraint verification methods concrete (script > manual)

## Review Process

1. Fork/submit your change
2. Verify it follows the format above
3. If adding a strategy: show it was tested on at least one task
4. If adding a trajectory: include the self-review verdict
