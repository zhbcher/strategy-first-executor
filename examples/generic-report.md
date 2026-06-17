# Example: Multi-source Research Report

A generic end-to-end example showing the full strategy → execute → gate → review loop.

## Task
> Research the current state of solid-state batteries, covering key players, technical challenges, and market outlook. Output a structured report.

## Phase 1: Strategy Output

```
## Strategy
### Goal
Produce a structured research report on solid-state batteries with three sections:
key players, technical challenges, market outlook.

### Constraints
- All claims must cite a source (URL or paper title)
- Report format: markdown with H2 sections, max 1500 words
- No speculation beyond cited sources

### Execution Plan
1. Research → web_search for each section topic, collect 3+ sources per section
   🔒 Checkpoint: have ≥3 relevant sources per section, all with URLs
2. Draft → write each section, cite sources inline
   🔒 Checkpoint: every claim has a citation, word count ≤1500
3. Polish → check formatting, fix broken citations, final read-through
   🔒 Checkpoint: valid markdown, no placeholder text, all URLs accessible

### Checkpoints
- After Phase 1: ≥3 sources per section
- After Phase 2: all claims cited, within word limit
- After Phase 3: markdown valid, no placeholders
```

## Phase 2 Checkpoint Gate (between Research and Draft)

```
PASS
```
→ 9 sources collected (3 per section), all with URLs. Proceeding to draft.

## Phase 3 Checkpoint Gate (between Draft and Polish)

```
FAIL: Section "Market Outlook" has 2 claims without citations.
```
→ Agent fixes citations, re-runs gate → PASS. Proceeds.

## Phase 3: Final Self-Review

```
## Final Self-Review

### Checkpoint Verification
- [x] Phase 1 — 9 sources, 3 per section
- [x] Phase 2 — all claims cited, 1420 words
- [x] Phase 3 — valid markdown, no placeholders

### Deviations
None

### Unresolved Issues
None

### Verdict
PASS
```
