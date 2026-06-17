# Strategy: Data Analysis Pipeline

## When to Use

Processing a dataset (CSV, JSON, Excel) through a structured analysis pipeline: ingest → clean → explore → model/insight → report. Typical tasks: "analyze this sales CSV and find trends", "explore user churn data and identify key factors", "process survey results and generate summary report".

## Typical Policy

```yaml
max_retry: 3
max_replan: 2
phase_timeout: 25m
max_total_runtime: 100m
```

## Strategy Template

```yaml
goal: "Analyze {dataset_description} from {data_file} and produce insights report with {N} key findings, supported by data"

constraints:
  - id: data_loaded
    type: script
    condition: "data file readable with expected column count"
    verify: exec: python -c "
import pandas as pd, sys
df = pd.read_csv('artifacts/cleaned_data.csv') if '{data_file}'.endswith('.csv') else pd.read_excel('artifacts/cleaned_data.csv')
print(f'Rows: {len(df)}, Cols: {len(df.columns)}')
assert len(df) > 0, 'Empty dataset'
"
  - id: no_nulls_critical
    type: metric
    condition: "critical columns have <5% nulls after cleaning"
    verify: exec: python -c "
import pandas as pd, json, sys
df = pd.read_csv('artifacts/cleaned_data.csv')
with open('artifacts/data_profile.json') as f:
    profile = json.load(f)
critical = profile.get('critical_columns', [])
for col in critical:
    null_pct = df[col].isnull().mean()
    if null_pct > 0.05:
        print(f'{col}: {null_pct:.1%} nulls (>5%)')
        sys.exit(1)
print('Critical columns within null threshold')
"
  - id: finding_evidence
    type: semantic
    condition: "each key finding in the report is backed by specific data or analysis result, not speculation"
    verify: manual

phases:
  - name: ingest
    actions: "Read {data_file}. Determine format, encoding, column types. Profile: row count, column names, dtypes, null percentages, basic statistics. Save data_profile.json."
    consume: []
    output: [data_profile.json]
    constraints:
      - id: profile_complete
        type: semantic
        condition: "data_profile.json covers all columns with type, null%, and basic stats"
        verify: manual

  - name: clean
    actions: "Based on data_profile.json: handle nulls (drop or fill), fix dtypes, remove duplicates, handle outliers (flag, don't silently drop unless extreme). Save cleaned_data.csv + cleaning_log.md (what was changed and why)."
    consume: [data_profile.json]
    output: [cleaned_data.csv, cleaning_log.md]
    constraints:
      - id: data_loaded
        type: script
        condition: "cleaned_data.csv is valid, non-empty"
        verify: exec: python -c "
import pandas as pd
df = pd.read_csv('artifacts/cleaned_data.csv')
assert len(df) > 0 and len(df.columns) > 0
print(f'OK: {len(df)} rows × {len(df.columns)} cols')
"
      - id: no_nulls_critical
        type: metric
        condition: "critical columns <5% nulls"
        verify: exec: python -c "
import pandas as pd, json, sys
df = pd.read_csv('artifacts/cleaned_data.csv')
with open('artifacts/data_profile.json') as f:
    profile = json.load(f)
for col in profile.get('critical_columns', []):
    if col in df.columns and df[col].isnull().mean() > 0.05:
        print(f'{col}: {df[col].isnull().mean():.1%} nulls')
        sys.exit(1)
print('OK')
"

  - name: explore
    actions: "Run exploratory analysis on cleaned_data.csv: distributions, correlations, trends over time (if datetime column), segmentations by key categories. Produce charts (save as PNGs) and exploration_notes.md summarizing patterns found."
    consume: [cleaned_data.csv, data_profile.json]
    output: [exploration_notes.md]
    constraints:
      - id: patterns_found
        type: semantic
        condition: "exploration_notes.md identifies at least 3 meaningful patterns or trends, not just summary statistics"
        verify: manual

  - name: analyze
    actions: "Based on exploration_notes.md, perform deeper analysis: statistical tests, regression, clustering, or whatever method fits the data and question. Produce analysis_results.json with methodology, results, and interpretation."
    consume: [cleaned_data.csv, exploration_notes.md]
    output: [analysis_results.json]
    constraints:
      - id: method_justified
        type: semantic
        condition: "chosen analysis methods are appropriate for the data type and question"
        verify: manual

  - name: report
    actions: "Compile all findings into report.md: executive summary, methodology, key findings (with evidence), visualizations, limitations, and recommendations. Every finding links back to analysis_results.json or exploration_notes.md."
    consume: [analysis_results.json, exploration_notes.md, data_profile.json]
    output: [report.md]
    constraints: [finding_evidence]
```

## Notes

- If data is too large for a single CSV load, add a `sample` phase before `ingest` with stratified sampling
- For time-series data, add constraints: "data spans at least N periods", "no gaps > X days"
- Common replan: dataset too messy → add more aggressive cleaning; insufficient patterns → broaden exploration scope
- If charts are critical, use a tool that can generate PNGs (matplotlib via exec, or browser-based charting)
- The `critical_columns` in data_profile.json should be set during ingest based on what matters for the analysis question
