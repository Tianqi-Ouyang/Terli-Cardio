# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

HARMONY Terlipressin **Cardio** sub-project — cardiopulmonary (echocardiographic) phenotyping of HRS-AKI patients treated with terlipressin. Shared context (dataset, derived variables, helper functions, coding conventions) lives in the parent `CLAUDE.md` one level up and applies here.

## Analysis Scope

Cohort is restricted to patients with at least one non-missing echo measurement among:

```
echo_ef, echo_earatio, echo_eeratio, echo_gls, echo_lateral_e
```

Six abnormal-echo indicators are derived in the analysis file (thresholds below). Tables are then stratified by the standard HRS outcomes.

| Variable | Abnormal cutoff |
|:---|:---|
| `echo_ef_abnormal` | `echo_ef <= 50` |
| `echo_eeratio_abnormal` | `echo_eeratio >= 15` |
| `echo_gls_abnormal` | `echo_gls < 18` |
| `echo_lateral_e_abnormal` | `echo_lateral_e < 10` |
| `echo_tr_abnormal` | `echo_tr > 2.8` |
| `echo_lavi_abnormal` | `echo_lavi > 34` |

Stratifications (Table 1 columns) per `Requests/Echo Data Request 01292016.docx`:

- `hrs_responders` (3-level: 0 non-responder, 1 partial, 2 complete)
- `hrs_responders_cat_2` (binary: any AKI-stage improvement)
- `time_to_death_status_90days`
- `terli_ae_respfail`

## Repository Layout

| Path | Description |
|:---|:---|
| `Code/cardio 2026.Rmd` | Current (Jan 2026) analysis — filters by any non-NA echo, derives abnormal flags, runs 5 Table 1s |
| `Code/Cardio.Rmd` | Older version — filters by `*_notavailable___999 == 0`, no abnormal flags |
| `Requests/Echo Data Request *.docx` | Analyst specification for the stratified tables |
| `Results/` | Rendered HTML + CSVs (`table_hrs_responders_*.csv`, `table_terli_ae_respfail.csv`, `table_time_to_death_status_90days.csv`) |
| `docs/` | Quarto website — `index.qmd`, `analysis.qmd`, symlinked variable dictionaries |

## Working with the Analysis

- The `.Rmd` files in `Code/` are the legacy analyst-facing source. The canonical analysis for the Quarto site is `docs/analysis.qmd` and **must** source the shared pipeline instead of reading xlsx directly:

  ```r
  source(knitr::purl(
    "/Users/to909/Desktop/Terlipressin projects/Terlipressin/code/shared_pipeline.qmd",
    quiet = TRUE
  ))
  # `master` is now loaded with every standard derived variable
  master <- master %>% filter(
    if_any(c(echo_ef, echo_earatio, echo_eeratio, echo_gls, echo_lateral_e), ~ !is.na(.))
  )
  ```

- **Do not** re-read `/Users/to909/Desktop/Terlipressin/final_master_0508.xlsx` (the path used in the old `.Rmd`s) — that file predates the cleaned master. Use `data/final_master_01282026.xlsx` from the main project.

- `create_table_one()` is provided by the shared pipeline. Local copies inside the `.Rmd` files are redundant; prefer the shared version so `nonnormal`, `exact`, `missing`, and `includeNA` behavior stays consistent across sub-projects.

## Rendering & Git

```bash
# Render the Quarto site (outputs to docs/_site/)
quarto render docs/

# Render a single page
quarto render docs/analysis.qmd
```

The GitHub remote for this sub-project is <https://github.com/Tianqi-Ouyang/Terli-Cardio.git>. The master xlsx is **not** committed here — it is tracked in the main Terlipressin repo and loaded by path from the shared pipeline.
