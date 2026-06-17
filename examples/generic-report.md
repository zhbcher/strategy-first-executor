# Example: Multi-source Research Report

Full end-to-end example with Run ID, Constraint DSL, Event Journal, and Artifact Chain.

## Task
> Research solid-state batteries: key players, technical challenges, market outlook. Output report.md.

## strategy.yaml

```yaml
goal: "Produce report.md on solid-state batteries with 3 sections, all claims cited, ≤1500 words"

constraints:
  - id: source_count
    type: metric
    condition: total_sources >= 9
    verify: exec: python -c "import json; d=json.load(open('.runs/run_20260617_120000/artifacts/research.json')); print(len(d['sources']))"

  - id: all_cited
    type: semantic
    condition: "every claim in the report has a source citation"
    verify: manual

  - id: word_limit
    type: metric
    condition: word_count <= 1500
    verify: exec: wc -w .runs/run_20260617_120000/artifacts/draft.md

  - id: has_sections
    type: regex
    condition: "document contains exactly 3 H2 sections"
    verify: exec: grep -c '^## ' .runs/run_20260617_120000/artifacts/final.md

  - id: no_placeholders
    type: semantic
    condition: "no placeholder text like TODO or TKTK anywhere in the document"
    verify: manual

phases:
  - name: research
    actions: "web_search for each section topic, collect 3+ sources per section, save to research.json"
    consume: []
    output: [research.json]
    constraints: [source_count]

  - name: draft
    actions: "read research.json, write draft.md with inline citations"
    consume: [research.json]
    output: [draft.md]
    constraints: [all_cited, word_limit]

  - name: polish
    actions: "fix formatting, verify URLs, final read-through, remove placeholders"
    consume: [draft.md]
    output: [final.md]
    constraints: [has_sections, no_placeholders]
```

## manifest.json

```json
{
  "run_id": "run_20260617_120000",
  "strategy": "strategy.yaml",
  "artifacts": ["research.json", "draft.md", "final.md"],
  "journal": "journal.jsonl",
  "created_at": "2026-06-17T12:00:00Z",
  "updated_at": "2026-06-17T12:00:00Z",
  "current_phase": null
}
```

## journal.jsonl (event-sourced)

```jsonl
{"event":"run_started","run_id":"run_20260617_120000","timestamp":"2026-06-17T12:00:00Z"}
{"event":"phase_started","run_id":"run_20260617_120000","phase":"research","timestamp":"2026-06-17T12:00:01Z"}
{"event":"artifact_created","run_id":"run_20260617_120000","phase":"research","path":"artifacts/research.json","timestamp":"2026-06-17T12:01:30Z"}
{"event":"checkpoint_passed","run_id":"run_20260617_120000","phase":"research","constraint":"source_count","evidence":"9","timestamp":"2026-06-17T12:01:31Z"}
{"event":"phase_completed","run_id":"run_20260617_120000","phase":"research","timestamp":"2026-06-17T12:01:31Z"}
{"event":"phase_started","run_id":"run_20260617_120000","phase":"draft","timestamp":"2026-06-17T12:01:32Z"}
{"event":"artifact_created","run_id":"run_20260617_120000","phase":"draft","path":"artifacts/draft.md","timestamp":"2026-06-17T12:05:00Z"}
{"event":"checkpoint_failed","run_id":"run_20260617_120000","phase":"draft","constraint":"word_limit","reason":"word_count=1620","timestamp":"2026-06-17T12:05:01Z"}
{"event":"artifact_created","run_id":"run_20260617_120000","phase":"draft","path":"artifacts/draft.md","timestamp":"2026-06-17T12:07:00Z"}
{"event":"checkpoint_passed","run_id":"run_20260617_120000","phase":"draft","constraint":"all_cited","evidence":"manual review passed","timestamp":"2026-06-17T12:07:01Z"}
{"event":"checkpoint_passed","run_id":"run_20260617_120000","phase":"draft","constraint":"word_limit","evidence":"1480","timestamp":"2026-06-17T12:07:02Z"}
{"event":"phase_completed","run_id":"run_20260617_120000","phase":"draft","timestamp":"2026-06-17T12:07:02Z"}
{"event":"run_completed","run_id":"run_20260617_120000","timestamp":"2026-06-17T12:10:00Z"}
```

Key events:
- `checkpoint_failed` on word_limit → draft trimmed → re-check passed
- All phases eventually completed
- Journal is replayable: you can reconstruct the entire execution timeline
