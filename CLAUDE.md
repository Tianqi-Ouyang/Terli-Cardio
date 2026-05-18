# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

HARMONY Terlipressin **Cardio** sub-project — cardiopulmonary (echocardiographic) phenotyping of HRS-AKI patients treated with terlipressin. Shared context (dataset, derived variables, helper functions, coding conventions) lives in the parent `CLAUDE.md` one level up and applies here.

## Analysis Scope (current spec, supersedes the older `Code/*.Rmd`)

Two cohorts:

- **Echo cohort** — at least one of `echo_ef_notavailable___999`, `echo_late_notavailable___999`, `echo_gls_notavailable___999`, `echo_earatio_notavailable___999`, `echo_eeratio_notavailable___999` equals 0 (i.e. ≥ 1 echo measurement available).
- **Pre-terli echo cohort** — echo cohort AND `days_to_echo < days_to_terli_initiation_day0`.

Eight outputs in `docs/analysis.qmd`:

| # | Output | Cohort | Stratifier (column) | Rows |
|:--|:--|:--|:--|:--|
| 1–3 | Counts of echo timing (before / same day / after terli) | echo cohort | — | — |
| 4 | Demographics Table 1 (~55 rows) | pre-terli | `hrs_responders` | age + 50+ categorical demographics |
| 5 | Echo Table 1 | pre-terli | `hrs_responders_cat_2` | echo_ef, echo_earatio, echo_eeratio, echo_gls, echo_lateral_e (all numeric) |
| 6 | Echo Table 1 | pre-terli | `terli_ae_respfail` | same 5 echo |
| 7 | Echo Table 1 | pre-terli | `time_to_death_status_90days` (capped at 90 days) | same 5 echo |
| 8 | Cubic-spline logistic regression | pre-terli | outcome `hrs_responders_cat_2` | `ns(echo_ef, df = 3)` + 95% CI plot |

The **abnormal-echo flag** definitions (EF < 65, \|GLS\| < 18, E/e′ > 15, lateral e′ < 10, TR > 2.8 m/s, LAVI > 34) are used by Tasks 5–7. The EF cutoff was updated 2026-05-14 from `< 60` to `< 65` (rounded up from the Youden-optimal 64.5 reported in Task 9). The older `Code/*.Rmd` files still show the legacy `< 50` / `< 60` thresholds and remain as analyst history.

## Repository Layout

| Path | Description |
|:---|:---|
| `docs/analysis.qmd` | Canonical analysis (tasks 1–8 above) — sources the shared pipeline |
| `docs/index.qmd` | Project summary page |
| `docs/_quarto.yml` | Site config |
| `docs/{redcap,derived}_variables.qmd` | Symlinks to the main project's variable dictionaries |
| `Code/cardio 2026.Rmd` | Older analyst version (abnormal-flag thresholds) — superseded |
| `Code/Cardio.Rmd` | Even older analyst version — superseded |
| `Requests/Echo Data Request *.docx` | Original analyst specification |
| `Results/` | Rendered HTML + CSVs |

## Data Caveats

- **`echo_gls` is harmonized to `abs(echo_gls)` in the `echo_cohort` step** — source data mixed two conventions (some centers store negative GLS, others store the absolute value; raw range was **−20 to +25**). `docs/analysis.qmd` takes `abs()` once at the cohort prep, so every downstream summary (Tasks 5–7 medians, the `echo_gls_abnormal` flag, splines) operates on magnitudes only. **83 % of the pre-terli cohort is still missing GLS**, so the medians remain n-limited.

## Sourcing the Shared Pipeline

```r
source(knitr::purl(
  "/Users/to909/Desktop/Terlipressin projects/Terlipressin/code/shared_pipeline.qmd",
  output = tempfile(fileext = ".R"),
  quiet  = TRUE
))
# `master` is now loaded with every standard derived variable
```

Pass `output = tempfile(...)` so we don't drop a stray `shared_pipeline.R` into
the working directory and parallel renders don't race on the same path.

**Do not** re-read `/Users/to909/Desktop/Terlipressin/final_master_0508.xlsx` (the
path used in the legacy `.Rmd`s) — that file predates the cleaned master. The
shared pipeline reads `data/final_master_01282026.xlsx` from the main project.

## Rendering & Git

```bash
# Render the Quarto site (outputs to docs/_site/)
quarto render docs/

# Render a single page
quarto render docs/analysis.qmd
```

Do **not** add a `Co-Authored-By: Claude ...` trailer to commit messages.

The GitHub remote is <https://github.com/Tianqi-Ouyang/Terli-Cardio.git>. The
master xlsx is **not** committed here — it is tracked in the main Terlipressin
repo and loaded by path from the shared pipeline.
