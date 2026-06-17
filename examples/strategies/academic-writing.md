# Strategy: Academic Paper Section

## When to Use

Writing a structured academic paper section (introduction, literature review, methodology, results) with proper citations from a provided reading list or research corpus. Supports Chinese (中文) and English output.

## Typical Policy

```yaml
max_retry: 3
max_replan: 2
phase_timeout: 30m
max_total_runtime: 120m
```

## Strategy Template

```yaml
goal: "Write the {section_name} section of an academic paper on {topic}, ≥{N} words, with proper citations from provided sources"

constraints:
  - id: citation_count
    type: metric
    condition: "at least {N} unique sources cited"
    verify: exec: python -c "
import re, sys
with open('artifacts/draft.md') as f:
    text = f.read()
cites = set(re.findall(r'\\cite\{([^}]+)\}', text) + re.findall(r'\[(\d+)\]', text))
actual = len(cites)
print(actual)
assert actual >= {N}, f'Expected ≥{N} unique citations, got {actual}'
"
  - id: section_structure
    type: semantic
    condition: "section has clear opening, body paragraphs with topic sentences, and concluding transition"
    verify: manual
  - id: no_unsupported
    type: semantic
    condition: "no factual claim without a citation, no hallucinated references"
    verify: manual
  - id: word_range
    type: metric
    condition: "{MIN} ≤ word_count ≤ {MAX}"
    verify: exec: |
      wc=$(wc -w < artifacts/draft.md | tr -d ' ')
      if [ "$wc" -lt {MIN} ] || [ "$wc" -gt {MAX} ]; then
        echo "word_count=$wc (expected {MIN}-{MAX})"
        exit 1
      fi
      echo "word_count=$wc OK"

phases:
  - name: source_analysis
    actions: "Read all provided papers/sources. For each, extract: key findings, methodology, relevance to topic, quotable passages. Save to source_notes.md with per-paper summaries."
    consume: []
    output: [source_notes.md]
    constraints:
      - id: sources_covered
        type: semantic
        condition: "every provided source has a summary entry, no source skipped"
        verify: manual

  - name: outline
    actions: "Based on source_notes.md, write a paragraph-level outline for the section. Each paragraph gets: topic sentence, which sources support it, transition to next. Save to outline.md."
    consume: [source_notes.md]
    output: [outline.md]
    constraints:
      - id: outline_complete
        type: semantic
        condition: "outline covers all major points from sources, logical flow between paragraphs"
        verify: manual

  - name: draft
    actions: "Write the full section following outline.md. Use proper academic style. Every factual claim cites specific sources from source_notes.md. Save to draft.md."
    consume: [source_notes.md, outline.md]
    output: [draft.md]
    constraints: [citation_count, section_structure, no_unsupported, word_range]

  - name: polish
    actions: "Read draft.md for style, flow, and citation formatting. Fix awkward sentences, ensure consistent citation style. Save polished version to final.md."
    consume: [draft.md]
    output: [final.md]
    constraints:
      - id: style_consistent
        type: semantic
        condition: "academic tone consistent throughout, citations uniformly formatted"
        verify: manual
```

## Notes

- This template assumes sources are already collected. If you need to find sources too, prepend a `literature_search` phase
- For Chinese academic writing, adjust the `citation_count` regex to match Chinese citation formats (e.g., `[1]`, `（张三，2024）`)
- Common replan: outline too ambitious for word limit → trim scope, focus on 2-3 strongest threads
- If you have a LaTeX target, add a `latex_build` constraint: "draft compiles without errors when inserted into main.tex"
