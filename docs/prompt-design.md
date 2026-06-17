# Prompt Design Rationale

Each prompt template serves a specific role in the execution loop. Here's why they're designed the way they are.

## strategy-prompt.md — Strategy Generation

**Role:** Called at run start and during replanning. Produces `strategy.yaml`.

**Key design choices:**

### Two modes, one prompt
The `REPLAN_CONTEXT_START/END` block is conditionally injected. When absent, it's a fresh strategy generation. When present, the prompt switches to replanning mode. One prompt, two behaviors — avoids drift between "new strategy" and "replan" logic.

### `[REPLAN ANALYSIS]` enforcement
The prompt doesn't just ask for analysis — it requires a specific comment block format. This makes replan analysis machine-parseable (future use) and prevents casual "I'll just tweak one word" replans that repeat the same failure.

### "Do NOT output a structurally identical strategy"
This is a hard gate. If the failure reason suggests the task is impossible, the prompt explicitly allows saying so — preventing infinite replan loops on unsolvable tasks.

### Constraint verification as part of strategy
Constraints aren't separate — they're embedded in the strategy YAML. This means `strategy.yaml` is the complete execution specification. You can hand it to another agent or version-control it as a unit.

## checkpoint-gate-prompt.md — Checkpoint Gate

**Role:** Called after every phase. Verifies artifacts, enforces policy, decides next action.

**Key design choices:**

### Protocol, not conversation
The prompt uses a numbered protocol (Step 0, 1, 2, 3) with explicit STOP commands. This prevents LLM "helpfulness" from skipping verification and "probably fine"-ing past failures.

### Policy check first (Step 0)
Policy violations are checked before any other work. If retries are exhausted or replan limit hit, the gate stops immediately — no wasted computation.

### Consume verification (Step 1)
Before checking phase output quality, verify the phase can even consume what it needs. This catches manifest/sync errors early.

### Three outcomes, not two
Most checkpoint designs have pass/fail. This one has pass/fail/FAIL_STRATEGY. FAIL_STRATEGY says "the approach is wrong, not just this attempt." Without this distinction, retry loops would waste attempts on unfixable strategies.

## self-review-prompt.md — Self Review

**Role:** Called after all phases complete. Audits the entire run.

**Key design choices:**

### Structure over narrative
The review follows a fixed template (Artifact Chain → Constraints → Policy → Deviations → Verdict). This ensures every run gets the same audit depth, regardless of task type.

### Deviations, not errors
"Deviations" captures actions that neither followed strategy nor advanced the task. This is broader than "bugs" — it catches LLM wandering, unnecessary steps, or context drift.

### Binary verdict
PASS or NEEDS_FIX. No "mostly done" or "good enough." If there's a deviation or unresolved issue, it's NEEDS_FIX. This forces hard decisions at the review boundary.

## Why These Three, Not More

Many task execution systems add more prompts over time (progress updates, human approval, parallel coordination). The Strategy-First Executor deliberately stays at three:

1. **Strategy** (plan) — once at start, maybe again on replan
2. **Checkpoint** (verify) — once per phase, maybe multiple on retry
3. **Self-Review** (audit) — once at end

Adding more prompts would blur the runtime's scope (ADR-004: no human layer) or create ambiguity about what each prompt owns. Three is the minimum viable set for a verifiable execution loop.
