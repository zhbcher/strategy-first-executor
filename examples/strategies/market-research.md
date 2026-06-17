# Strategy: Market Research Report

## When to Use

Researching a market, industry, or competitive landscape and producing a structured report with cited sources. Typical tasks: "analyze the EV battery market", "competitive landscape of AI code editors", "market sizing for plant-based meat in APAC".

## Typical Policy

```yaml
max_retry: 3
max_replan: 2
phase_timeout: 25m
max_total_runtime: 90m
```

## Strategy Template

```yaml
goal: "Research {topic} and produce a structured report with ≥{N} cited sources, covering {sections}"

constraints:
  - id: source_count
    type: metric
    condition: "total unique sources ≥{N}"
    verify: exec: python -c "
import json
d = json.load(open('artifacts/research.json'))
sources = {s['url'] for section in d['sections'] for s in section.get('sources', [])}
actual = len(sources)
print(actual)
assert actual >= {N}, f'Expected ≥{N} sources, got {actual}'
"
  - id: section_coverage
    type: semantic
    condition: "report covers all required sections: {sections}"
    verify: manual
  - id: all_cited
    type: semantic
    condition: "every factual claim in the report has a source citation"
    verify: manual
  - id: word_limit
    type: metric
    condition: "report ≤{M} words"
    verify: exec: wc -w artifacts/draft.md

phases:
  - name: research
    actions: "For each section, web search and collect sources. Record title, URL, key findings, publication date. Save structured research to research.json with sections array."
    consume: []
    output: [research.json]
    constraints: [source_count]

  - name: draft
    actions: "Read research.json. Write a structured report in draft.md: executive summary, then one section per required topic. Every factual claim must cite its source (Author, Year) or [Source URL]. No unsupported assertions."
    consume: [research.json]
    output: [draft.md]
    constraints: [section_coverage, all_cited, word_limit]

  - name: verify
    actions: "Read draft.md. For each section, spot-check 3 claims against their cited sources. Check for hallucinated facts or misattributions. Produce verification.md with findings."
    consume: [draft.md, research.json]
    output: [verification.md]
    constraints:
      - id: no_hallucinations
        type: semantic
        condition: "spot-checked claims match their cited sources, no fabricated data"
        verify: manual

  - name: finalize
    actions: "Apply verification fixes to draft.md. Add a sources appendix listing all citations. Save as final.md."
    consume: [draft.md, verification.md]
    output: [final.md]
    constraints:
      - id: has_appendix
        type: regex
        condition: "final.md contains a sources/appendix section"
        verify: exec: grep -q "## Sources\|## Appendix\|## References" artifacts/final.md
```

## Notes

- If research phase returns too few sources, replan: broaden search terms or merge thin sections
- The `verify` phase is the anti-hallucination gate — don't skip it
- For time-sensitive topics (market data, stock prices), add a `freshness` constraint: "no source older than 6 months"
- If the topic is extremely niche, set `source_count: 4-6` and flag in self-review
