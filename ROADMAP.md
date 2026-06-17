# Roadmap

## v7.1 — Ecosystem (current)

- [x] Recovery Protocol + Replan Reflection (v7 core)
- [ ] Strategy template library: 5+ domains covered
- [ ] Execution trajectory examples: successful run + crash recovery
- [ ] README, CONTRIBUTING, ROADMAP
- [ ] `docs/`: architecture, prompt design, FAQ

## v7.2 — Validation

- [ ] Benchmark task suite (3-5 standardized tasks)
- [ ] Success criteria per task (qualitative + quantitative)
- [ ] Automated benchmark runner script
- [ ] Version-vs-version comparison report
- [ ] First benchmark results published

## v8 Candidates

Under evaluation. No commitment yet.

### Checkpoint Branching
Allow a checkpoint to spawn a sub-strategy when unexpected data is found.
- **Risk:** Breaks linear execution model. Journal replay, resume protocol all need redesign.
- **Benefit:** Flexible execution paths for exploratory tasks.
- **Status:** Researching. Need ADR.

### Strategy A/B Testing
Generate two strategies, execute in parallel, compare results.
- **Risk:** LLM non-determinism makes "which is better" judgment noisy.
- **Benefit:** Could optimize strategy design over time.
- **Status:** Deferred. Requires a Meta-Runtime layer outside current scope.

### External Knowledge Integration
Allow phases to pull from configured knowledge bases (wiki, vector stores).
- **Risk:** Adds dependency on external systems.
- **Benefit:** Grounded execution for research-heavy tasks.
- **Status:** Exploring Constraint extension (`type: knowledge`).

## Non-Goals (explicitly out of scope)

Per ADR-004, the Task Runtime (L1) does not own the human layer:

- ❌ Approval workflows
- ❌ User notifications
- ❌ Skip/escalate/pause
- ❌ UI/dashboard
- ❌ Permission management

These belong to the Human Layer (L3) or Platform Layer (L0).
