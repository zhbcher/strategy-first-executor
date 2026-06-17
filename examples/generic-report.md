# Example: Multi-source Research Report

A generic end-to-end example showing full strategy → artifact → gate → journal loop.

## Task
> Research solid-state batteries: key players, technical challenges, market outlook. Output report.md.

## Phase 1: Strategy Output

```
## Strategy
### Goal
Produce report.md on solid-state batteries (3 sections, all claims cited, ≤1500 words)

### Constraints
- All claims must cite a source → Verify: manual
- Word count ≤ 1500 → Verify: exec: wc -w report.md

### Execution Plan

#### Phase: research
Actions: web_search for each section topic, collect 3+ sources per section
Input: null
Output: research.json
🔒 Checkpoint:
  metric: source_count
  operator: ">="
  value: 9
  verify: exec: python -c "import json; d=json.load(open('research.json')); print(len(d['sources']))"

#### Phase: draft
Actions: read research.json, write draft.md with citations
Input: research.json
Output: draft.md
🔒 Checkpoint:
  condition: every claim has a citation, word count ≤ 1500
  verify: manual

#### Phase: polish
Actions: fix formatting, check URLs, final read-through
Input: draft.md
Output: report.md
🔒 Checkpoint:
  condition: valid markdown, no placeholders, all URLs accessible
  verify: manual
```

## execution_state.json (after Phase 1)

```json
{
  "task": "Research solid-state batteries",
  "strategy": "...",
  "phases": {
    "research": {"status": "passed", "artifact": "research.json", "updated_at": "2026-06-17T12:00:00Z"},
    "draft": {"status": "running", "artifact": null, "updated_at": "2026-06-17T12:00:01Z"},
    "polish": {"status": "pending", "artifact": null, "updated_at": null}
  }
}
```

## execution_journal.jsonl

```jsonl
{"phase":"research","action":"web_search","result":"success","evidence":"5 results","timestamp":"2026-06-17T11:59:00Z"}
{"phase":"research","action":"web_search","result":"success","evidence":"4 results","timestamp":"2026-06-17T11:59:30Z"}
{"phase":"research","action":"checkpoint_gate","result":"pass","evidence":"source_count=9","timestamp":"2026-06-17T12:00:00Z"}
{"phase":"draft","action":"read_artifact","result":"success","evidence":"research.json loaded","timestamp":"2026-06-17T12:00:05Z"}
```

## Checkpoint Gate (research → draft)

```
PASS
```
→ Tool output: `9`. ≥9 → pass. Journal appended. State updated. Proceed to draft.