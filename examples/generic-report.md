# Example: Multi-source Research Report

Full end-to-end example with Policy, Consume Verification, Constraint DSL, Event Journal.

## Task
> Research solid-state batteries: key players, technical challenges, market outlook. Output report.md.

## policy.yaml

```yaml
max_retry: 3
max_replan: 2
phase_timeout: 30m
max_total_runtime: 120m
```

## strategy.yaml

```yaml
goal: "Produce report.md on solid-state batteries with 3 sections, all claims cited, ≤1500 words"

constraints:
  - id: source_count
    type: metric
    condition: total_sources >= 9
    verify: exec: python -c "import json; d=json.load(open('artifacts/research.json')); print(len(d['sources']))"
  - id: all_cited
    type: semantic
    condition: "every claim has a source citation"
    verify: manual
  - id: word_limit
    type: metric
    condition: word_count <= 1500
    verify: exec: wc -w artifacts/draft.md

phases:
  - name: research
    actions: "web_search for each section, collect 3+ sources per section"
    consume: []
    output: [research.json]
    constraints: [source_count]
  - name: draft
    actions: "read research.json, write draft.md with inline citations"
    consume: [research.json]
    output: [draft.md]
    constraints: [all_cited, word_limit]
  - name: polish
    actions: "fix formatting, verify URLs, final read-through"
    consume: [draft.md]
    output: [final.md]
    constraints: []
```

## journal.jsonl

```jsonl
{"event":"run_started","run_id":"run_20260617_120000","timestamp":"2026-06-17T12:00:00Z"}
{"event":"phase_started","run_id":"run_20260617_120000","phase":"research","timestamp":"2026-06-17T12:00:01Z"}
{"event":"consume_verified","run_id":"run_20260617_120000","phase":"research","artifacts":[],"timestamp":"2026-06-17T12:00:01Z"}
{"event":"artifact_created","run_id":"run_20260617_120000","phase":"research","path":"artifacts/research.json","timestamp":"2026-06-17T12:01:30Z"}
{"event":"checkpoint_passed","run_id":"run_20260617_120000","phase":"research","constraint":"source_count","evidence":"9","timestamp":"2026-06-17T12:01:31Z"}
{"event":"phase_completed","run_id":"run_20260617_120000","phase":"research","timestamp":"2026-06-17T12:01:31Z"}
{"event":"phase_started","run_id":"run_20260617_120000","phase":"draft","timestamp":"2026-06-17T12:01:32Z"}
{"event":"consume_verified","run_id":"run_20260617_120000","phase":"draft","artifacts":["research.json"],"timestamp":"2026-06-17T12:01:32Z"}
{"event":"artifact_created","run_id":"run_20260617_120000","phase":"draft","path":"artifacts/draft.md","timestamp":"2026-06-17T12:05:00Z"}
{"event":"checkpoint_failed","run_id":"run_20260617_120000","phase":"draft","constraint":"word_limit","reason":"word_count=1620","timestamp":"2026-06-17T12:05:01Z"}
{"event":"artifact_created","run_id":"run_20260617_120000","phase":"draft","path":"artifacts/draft.md","timestamp":"2026-06-17T12:07:00Z"}
{"event":"checkpoint_passed","run_id":"run_20260617_120000","phase":"draft","constraint":"all_cited","evidence":"manual review passed","timestamp":"2026-06-17T12:07:01Z"}
{"event":"checkpoint_passed","run_id":"run_20260617_120000","phase":"draft","constraint":"word_limit","evidence":"1480","timestamp":"2026-06-17T12:07:02Z"}
{"event":"phase_completed","run_id":"run_20260617_120000","phase":"draft","timestamp":"2026-06-17T12:07:02Z"}
{"event":"run_completed","run_id":"run_20260617_120000","timestamp":"2026-06-17T12:10:00Z"}
```

## Failure Scenarios

### Consume Verification Failure

```jsonl
{"event":"consume_validation_failed","run_id":"run_20260617_120000","phase":"draft","missing":["research.json"],"timestamp":"2026-06-17T12:01:32Z"}
```
→ Phase 2 never starts. Run terminates immediately.

### Policy Violation (max_retry)

```jsonl
{"event":"policy_violation","run_id":"run_20260617_120000","phase":"draft","violation":"max_retry","limit":3,"actual":3,"timestamp":"2026-06-17T12:10:00Z"}
```
→ Run terminates. No infinite retry loop.
