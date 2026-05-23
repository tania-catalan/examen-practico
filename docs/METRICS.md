Metrics for prompt runs

This repository includes prompt templates under `docs/prompts/` that will be executed repeatedly. To measure quality improvements across executions we track a set of metrics per document type (PRD, FSD, C4). Below is a compact reference of the metrics, the calculation logic and suggested thresholds.

### Metrics summary (per run, per document)

| Metric | PRD | FSD | C4 (N1/N2) | Calculation / Notes |
|---|---:|---:|---:|---|
| Completitud del output (%) | ✓ | ✓ | ✓ | 100 * (sections_found / sections_expected)
| % Secciones cubiertas | ✓ | ✓ | ✓ | same as completitud
| Cobertura NFRs (%) | ✓ | — | — | 100 * (nfrs_cited / nfrs_expected)
| UCs entregados (count) | — | ✓ | — | number of complete UCs (target e.g. 5)
| % UCs con Given/When/Then | — | ✓ | — | 100 * (uc_with_gwt / uc_total)
| Trazabilidad (%) | ✓ | ✓ | ✓ | 100 * (items_traced / items_total)
| Sintaxis Mermaid válida | — | — | ✓ | boolean / mermaid validator
| Reducción de iteraciones (%) | ✓ | ✓ | ✓ | 100 * (it_start - it_current) / it_start
| Tiempo hasta convergencia (min) | ✓ | ✓ | ✓ | runtime until document passes checks
| Ediciones humanas (count) | ✓ | ✓ | ✓ | human commits/line-diffs after generation
| Inventado / Hallucination (%) | ✓ | ✓ | ✓ | 100 * (invented_items / total_items) (penaliza)

### Suggested thresholds

- completeness >= 90%
- traceability = 100% for ADRs, >= 90% for PRD/FSD
- nfr_coverage >= 80%
- uc_gwt_pct = 100% (each UC should include Given/When/Then)
- mermaid_valid = true for C4 diagrams
- invented_pct <= 5%

### How to compute (short formulas)

- sections_found: count Markdown headings that match expected section titles (use canonical list per doc type).
- nfrs_cited: detect explicit citation patterns or exact NFR labels from `docs/contexto.md`.
- uc_with_gwt: parse UCs and check for presence of 'Given', 'When', 'Then' tokens.
- items_traced: count occurrences of citations to book/brief/US in decisions or claims.
- it_start / it_current: iterations as measured by prompt run metadata (or number of prompt runs until acceptance).

### Quick PowerShell examples (Windows PowerShell v5.1)

Count headings (approximate sections_found):

```powershell
Select-String -Path docs/prompts/PRD_v1.md -Pattern '^(#{1,3})\s+' -AllMatches | Measure-Object | Select-Object -ExpandProperty Count
```

Detect traceability citations (book / brief / US):

```powershell
Select-String -Path docs/prompts/ADR_v1.md -Pattern 'Microservices Patterns|\[Brief|US-0[1-3]' -AllMatches | Measure-Object | Select-Object -ExpandProperty Count
```

Detect Given/When/Then count per UC (heuristic):

```powershell
# Count occurrences of Given/When/Then in FSD
+(Select-String -Path docs/prompts/FSD_v1.md -Pattern 'Given:|When:|Then:' -AllMatches).Matches.Count
```

### Notes and next steps

- For robust automation we recommend a small validator script (PowerShell/Python) that reads a JSON with the expected sections per document and computes all metrics, storing results in `metrics_runs.csv`.
- If you want, I can: generate the JSON of expected sections, implement the PowerShell script and run it once against the `_v1.md` files to produce a sample `metrics_runs.csv`.
