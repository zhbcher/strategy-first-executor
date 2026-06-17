# Architecture Decision Records

Project: strategy-first-executor
Format: ADR (Architecture Decision Record)

---

# ADR-001: Artifact Is The Single Source Of Truth

Date: 2026-06-17
Status: Accepted

## Context

An earlier design stored execution state in `execution_state.json` alongside artifact files. This created a dual source of truth: state could say "done" while the artifact file was missing, or vice versa.

## Decision

Artifact files are the single source of truth for phase completion. State is dynamically derived from artifact existence, never stored separately. `manifest.json` is an index — it points to artifacts but does not duplicate their state.

## Consequences

- Prohibited: storing phase status in any file other than artifacts themselves
- Required: consume verification must check both manifest registration AND filesystem existence
- Required: resume logic derives current phase from the journal's last `phase_completed` event, not from a stored state field

---

# ADR-002: Policy Is A Governance Primitive, Not A Core Primitive

Date: 2026-06-17
Status: Accepted

## Context

Policy (retry limits, timeouts, replan bounds) was initially treated as a peer to the five core primitives (Run, Strategy, Artifact, Constraint, Event). This created confusion: Policy does not generate artifacts, does not emit events, and does not participate in the execution data flow.

## Decision

The system has 5 Core Primitives (data objects) and 1 Governance Primitive (controller):

Core: Run, Strategy, Artifact, Constraint, Event
Governance: Policy

Policy wraps the runtime. It is not part of the data flow.

## Consequences

- Policy must not generate artifacts
- Policy must not write journal events (violations are logged by the guard, not by Policy itself)
- Policy fields are limited to stopping conditions (retry, timeout, replan) — never approval, notification, or UI concerns

---

# ADR-003: No Seventh Primitive

Date: 2026-06-17
Status: Accepted

## Context

The six primitives (5 Core + 1 Governance) have stabilized. Adding more primitives risks conceptual inflation.

## Decision

No seventh primitive will be added. New capabilities must attach to an existing primitive or be implemented as a subordinate module (Adapter, Plugin, Guard, Registry).

## Consequences

- Allowed: adding fields to existing primitives
- Allowed: adding Guard modules that operate within existing primitives
- Prohibited: creating a new top-level primitive

---

# ADR-004: Runtime Does Not Own The Human Layer

Date: 2026-06-17
Status: Accepted

## Context

Human interaction concerns (approval, notification, skip, escalation) were proposed as Runtime features. These belong to a separate architectural layer.

## Decision

The Task Runtime (L1) is bounded by the Runtime Contract v1. Human interaction (L3) is out of scope. The Runtime may stop and report failure; it may not prompt, notify, or wait for human input.

## Consequences

OUT OF SCOPE: Approval, Notification, UI, Memory, Permission, Skip, Escalate
IN SCOPE: Task Decomposition, Strategy, Execution, Artifact, Constraint, Journal, Recovery, Resume, Policy

---

# ADR-005: Consume Must Be Verified, Not Just Declared

Date: 2026-06-17
Status: Accepted

## Context

Phases declare `consume` artifacts in strategy.yaml, but no mechanism enforced that these artifacts actually exist before the phase begins. A missing input artifact could cause silent failure.

## Decision

Before every phase execution, the Runtime must verify:
1. Every consume file is registered in manifest.json's artifact list
2. Every consume file exists on the filesystem

Both conditions must pass. Failure emits `consume_validation_failed` and stops the run.

## Consequences

- Required: consume verification step before every phase (no skip)
- Required: consume_validation_failed is a terminal event
- Required: manifest must list all artifacts that any phase may consume
