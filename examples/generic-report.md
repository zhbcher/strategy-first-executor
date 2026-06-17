# Example: Multi-source Research Report

Full end-to-end example with Policy, Consume Verification, Replan Reflection, and Event Journal.

## Task
> Research solid-state batteries: key players, technical challenges, market outlook. Output report.md.

## policy.yaml

```yaml
max_retry: 3
max_replan: 2
phase_timeout: 30m
max_total_runtime: 120m
```

## strategy.yaml (first attempt)

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
    actions: "web_search 3+ sources per section"
    consume: []
    output: [research.json]
    constraints: [source_count]
  - name: draft
    actions: "read research.json, write draft.md"
    consume: [research.json]
    output: [draft.md]
    constraints: [all_cited, word_limit]
  - name: polish
    actions: "fix formatting, final read-through"
    consume: [draft.md]
    output: [final.md]
    constraints: []
```

## strategy.yaml (replan — after research phase found zero sources)

```yaml
# [REPLAN ANALYSIS]
# Why it failed: Phase 'research' found zero relevant sources for "market outlook".
#   The topic is too niche for current search tools.
# What changed: Merged "market outlook" expectations into the other two sections.
#   Added constraint 'min_sources' with relaxed threshold (6 instead of 9).
#   Phase 'research' now explicitly searches broader queries.
---
goal: "Produce report.md on solid-state batteries with 2 sections (key players + technical challenges), all claims cited, ≤1200 words"

constraints:
  - id: min_sources
    type: metric
    condition: total_sources >= 6
    verify: exec: python -c "import json; d=json.load(open('artifacts/research.json')); print(len(d['sources']))"
  - id: all_cited
    type: semantic
    condition: "every claim has a source citation"
    verify: manual

phases:
  - name: research
    actions: "web_search with broader queries: 'solid-state battery companies 2026', 'solid-state battery technical challenges'"
    consume: []
    output: [research.json]
    constraints: [min_sources]
  - name: draft
    actions: "read research.json, write draft.md"
    consume: [research.json]
    output: [draft.md]
    constraints: [all_cited]
  - name: polish
    actions: "fix formatting, final read-through"
    consume: [draft.md]
    output: [final.md]
    constraints: []
```

## journal.jsonl (with crash + resume)

```jsonl
{"event":"run_started","run_id":"run_20260617_120000","timestamp":"2026-06-17T12:00:00Z"}
{"event":"phase_started","run_id":"run_20260617_120000","phase":"research","timestamp":"2026-06-17T12:00:01Z"}
{"event":"consume_verified","run_id":"run_20260617_120000","phase":"research","artifacts":[],"timestamp":"2026-06-17T12:00:01Z"}
{"event":"artifact_created","run_id":"run_20260617_120000","phase":"research","path":"artifacts/research.json","timestamp":"2026-06-17T12:01:30Z"}
{"event":"checkpoint_passed","run_id":"run_20260617_120000","phase":"research","constraint":"min_sources","evidence":"6","timestamp":"2026-06-17T12:01:31Z"}
{"event":"phase_completed","run_id":"run_20260617_120000","phase":"research","timestamp":"2026-06-17T12:01:31Z"}
{"event":"phase_started","run_id":"run_20260617_120000","phase":"draft","timestamp":"2026-06-17T12:01:32Z"}
{"event":"consume_verified","run_id":"run_20260617_120000","phase":"draft","artifacts":["research.json"],"timestamp":"2026-06-17T12:01:32Z"}
// --- CRASH here: system killed at 12:03:00 ---
// On restart, journal scanned:
//   Last phase_completed = research (anchor)
//   Target Resume Phase = draft
//   checkpoint_failed events in draft phase = 0 → retry_count = 0
//   Partial draft.md ignored → will be regenerated
//   Resume from Step A (Consume Verification) of draft phase
{"event":"phase_started","run_id":"run_20260617_120000","phase":"draft","timestamp":"2026-06-17T12:10:00Z"}
{"event":"consume_verified","run_id":"run_20260617_120000","phase":"draft","artifacts":["research.json"],"timestamp":"2026-06-17T12:10:01Z"}
{"event":"artifact_created","run_id":"run_20260617_120000","phase":"draft","path":"artifacts/draft.md","timestamp":"2026-06-17T12:13:00Z"}
{"event":"checkpoint_passed","run_id":"run_20260617_120000","phase":"draft","constraint":"all_cited","evidence":"manual review passed","timestamp":"2026-06-17T12:13:01Z"}
{"event":"phase_completed","run_id":"run_20260617_120000","phase":"draft","timestamp":"2026-06-17T12:13:01Z"}
{"event":"run_completed","run_id":"run_20260617_120000","timestamp":"2026-06-17T12:15:00Z"}
```
