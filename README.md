# Strategy-First Executor (Task Runtime)

> A recoverable, verifiable, resumable Task Runtime for multi-phase agentic execution on OpenClaw.
> Inspired by the [StraTA framework](https://arxiv.org/abs/2605.06642).

**v7** — Recovery Protocol + Replan Reflection. Production-ready foundation.

---

## What It Does

Turns complex multi-step AI tasks into structured, auditable, crash-proof runs. Instead of one long prompt, each task gets:

```
Strategy (what to do)
  ↓
Phase 1 → Artifact → Checkpoint ✓
Phase 2 → Artifact → Checkpoint ✓
Phase 3 → Artifact → Self-Review ✓
```

If something fails mid-way or the system restarts, it picks up where it left off — no lost work.

## Why Use It

| Without Runtime | With Runtime |
|---|---|
| One giant prompt, easy to lose context | Structured phases with artifact boundaries |
| Fail at step 4 → restart from step 1 | Crash at phase 2 → resume from phase 2 |
| "Did it actually do what I asked?" | Every phase has verifiable constraints |
| LLM drifts, hallucinates progress | Event journal records what actually happened |
| Same task 3 times, 3 different approaches | Strategy is explicit, auditable, reproducible |

## Quick Start

```
User: "Research solid-state battery market and write a report"

Runtime:
  1. Generates strategy.yaml (3 phases: research → draft → polish)
  2. Phase 1: web search, produce research.json, verify ≥6 sources
  3. Phase 2: read research.json, write draft.md, verify all claims cited
  4. Phase 3: polish, produce final.md
  5. Self-review: artifact chain check, constraint summary, verdict

Output: .runs/run_20260617_120000/artifacts/final.md
        .runs/run_20260617_120000/journal.jsonl (full audit trail)
```

## Key Features (v7)

- **🔄 Recovery Protocol** — System crash mid-phase? Journal replay finds safe anchor, restores counters, resumes cleanly
- **🔁 Replan Reflection** — Strategy failed? `[REPLAN ANALYSIS]` block required in new strategy: why it failed + what changed
- **✅ Consume Verification** — Every phase verifies its inputs exist (manifest + filesystem) before executing
- **🛡️ Policy Enforcement** — Retry limits, replan caps, timeouts. Policy only stops; never generates.
- **📋 Event Journal** — Every action emits a journal event. State is derived, never stored twice.

## Architecture

```
OpenClaw
  ↓
Task Runtime (strategy-first-executor)     ← this project
  ↓
Business Skills (video, research, etc.)
```

**5 Core Primitives + 1 Governance:**
- Run — isolated execution instance
- Strategy — what to execute
- Artifact — what was produced (single source of truth)
- Constraint — typed verification (metric | semantic | script | regex | tool)
- Event — what happened (journal.jsonl)
- Policy — runtime bounds (retry, timeout, replan)

## Examples

| File | What It Shows |
|---|---|
| `examples/video-production.md` | Video workflow: script → TTS → HTML → render |
| `examples/generic-report.md` | Research report with crash recovery + replan |
| `examples/strategies/` | Strategy templates for common domains |
| `examples/trajectories/` | Full execution logs (best practice reference) |

## Documentation

- [Architecture Design](docs/architecture.md) — ADRs, primitives, contracts
- [Prompt Design](docs/prompt-design.md) — Why each prompt template works the way it does
- [FAQ](docs/faq.md) — Common questions and edge cases
- [Roadmap](ROADMAP.md) — Where this is going
- [Contributing](CONTRIBUTING.md) — How to help

## References

- StraTA paper: [arxiv.org/abs/2605.06642](https://arxiv.org/abs/2605.06642)
- ADRs: `references/decisions.md`
