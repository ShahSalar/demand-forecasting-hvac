# Project Charter — LA HVAC Demand Forecasting (v1)

**Owner:** Sepehr Kermanshah
**Created:** 2026-08-26
**Status:** scope locked, build not started

---

## 1. Problem statement

Forecast weekly HVAC/mechanical permit volume in the City of Los Angeles, six weeks
ahead, as a demand signal for the residential HVAC trade.

This is v1, built on public data. A later version (v2) will run the same system on
OceanAirFlow's own operational job data once enough clean history has been collected.

**What this measures:** installation and replacement activity across every contractor
in LA City, as recorded by permit filings.

**What this does not measure:** service and repair calls (no permit required), work
performed without a permit, and any single company's workload. The date on a record is
the date the permit was **issued** by the city, which leads the start of work by an
unknown and possibly variable lag.

---

## 2. The ten scope decisions

### 2.1 Primary dataset

**LADBS Mechanical Permits Issued**, published on data.lacity.org.

The single combined permit dataset (`yv23-pmwf`) was retired on 2023-05-22 and replaced
by a set split three ways: by permit category (Building / Electrical / Mechanical), by
Submitted vs Issued, and by era (Before 2010 / Between 2010 and 2019 / 2020 to Present).

**Use Issued, not Submitted.** A permit that was filed but never issued is not work, so
Issued is the correct demand signal.

**Default to two eras: 2010–2019 and 2020–Present.** That is roughly 860 weeks, which is
ample, and it avoids both the pre-2010 schema seam and the 2008–09 construction collapse
as a structural break in training data. Adding the Before-2010 table is optional and
should only happen for a stated reason, logged.

Dataset IDs are not recorded here on purpose — find them yourself in the catalog and
write them into the loader as named constants with a comment giving each dataset's full
title, since these get re-versioned.

**"Mechanical" is broader than HVAC.** The category covers plumbing, HVAC systems, fire
sprinklers, elevators, and pressure vessels. Filtering down to HVAC is a second step
inside the mechanical table, on whatever work-description or permit-subtype field exists.
Assessing that field's vocabulary is a data checkpoint deliverable.

Confirmation pending: the data checkpoint verifies the series has sufficient volume, no
catastrophic gaps, and visible seasonality. If it fails, fall back to EIA-930 hourly
electricity load for the CAISO/LADWP region.

### 2.2 Target variable
**Weekly count of HVAC/mechanical permits issued.** One target for v1.

Permit valuation (dollars) is deferred to stage 5, conditional on the field carrying
real values rather than flat-fee placeholders. Verify at the data checkpoint: what
fraction are zero or null, is the distribution natural or clustered on round numbers,
does average valuation per permit move sensibly across 15 years.

### 2.3 Granularity
**Weekly. Monday-start weeks, labeled by the Sunday end** (`resample("W-SUN")`).

Rationale: LADBS issues permits Monday–Friday, so any convention preserving an intact
Mon–Fri block behaves identically. Monday-start matches ISO week numbering for clean
joins to weather data, and OceanAirFlow's Mon–Sat work week sits entirely inside a
Mon–Sun bucket, so v1 and v2 share a convention.

Partial weeks at the start and end of the series are dropped.

Daily granularity is out of scope.

### 2.4 Horizon
**Six weeks ahead.**

> Six weeks matches the operational planning horizon in residential HVAC — equipment
> ordering lead times, the hiring and onboarding cycle for an additional tech, and the
> lag between permit filing and work commencing. It is long enough to inform a decision
> and short enough that recent-activity signals remain informative.

Error is reported at every horizon from 1 to 6. The headline claim is stated at week 4.

### 2.5 Hierarchy
v1 forecasts the **citywide total only** — a single series.

Geographic (ZIP, council district) and subtype (residential/commercial, new/alteration)
columns are preserved through the pipeline but not modeled in v1.

Stage 6 splits the total, subtype first, geography second. A grouping is modeled only
if every resulting series averages **≥15 permits per week** over the full history.
Two-dimensional grouping (geography × subtype) is permanently out of scope.

### 2.6 Success criterion

**Primary:** MASE ≤ 0.85 versus seasonal-naive at 4-week horizon, on the rolling-origin
backtest. (MASE defined as model MAE divided by seasonal-naive MAE over identical
evaluation windows — state this definition explicitly in the write-up.)

**Secondary:** 90% prediction intervals achieve 85–95% empirical coverage at week 4.

**Reporting:** MASE and coverage reported at every horizon 1 through 6. The result is
reported whether or not the target is met.

**Revision:** one adjustment permitted after computing the seasonal-naive baseline and
examining the series, but before fitting any real model. Logged and dated in the
decision log. Fixed thereafter.

### 2.7 Backtest design

Rolling-origin backtest:

- **Training window:** expanding (each fold trains on all history up to its cutoff)
- **Folds:** 12, stepped 4 weeks apart
- **Gap:** none — permit records post promptly, so no simulated data delay is needed
- **Holdout:** final 26 weeks reserved and untouched during development, evaluated
  exactly once at project completion on the pre-committed final model
- **Fairness:** the seasonal-naive baseline and every candidate model are scored on
  identical folds with identical metrics

Each fold fits a fresh model from scratch. Nothing carries between folds. The backtest
measures; it does not improve the model.

**Revision:** permitted only for a stated methodological reason (e.g. a structural break
making old data unrepresentative), logged and dated, applied uniformly to baseline and
all models. Revision on the basis of observed model scores is not permitted.

### 2.8 Timeline

v1 complete by **December 15, 2026**, at roughly 10 hrs/week.

Front-loaded light: 5–6 hrs/week through mid-October (peak cooling season for the
business), 12–14 hrs/week in November.

### 2.9 Repository posture

**Public on GitHub from the first commit.** v1 uses entirely public data, so there is
nothing to protect, and a visible commit history is a record of sustained work.

`.gitignore` excludes `.venv/`, `__pycache__/`, and raw data files. Raw CSVs are
re-downloadable; commit the fetching code instead. No credentials, ever.

README written as a stub in week one and updated throughout: what the project does,
what data, what was found, what the limitations are — in that order.

### 2.10 Scope boundaries

**Out of scope, permanently:**
Lead scoring. Price optimization. Natural-language query interface. Automated scheduling
or crew assignment. Real-time or streaming data. Two-dimensional hierarchical splits.
Daily granularity. Deployment for external users. Any model type outside the five
comparison arms.

**Deferred with stated conditions:**
- Valuation as second target — stage 5, conditional on field quality
- Subtype hierarchy — stage 6, conditional on ≥15 permits/week per series
- Geographic hierarchy — stage 6, after subtype, same threshold
- Hierarchical reconciliation — only after independent forecasts at each level work
- AHRI national shipments as covariate — any time after baseline
- Submitted-vs-issued gap as a leading indicator — the Submitted datasets exist
  alongside Issued; the lag between them may carry signal. Recorded, not pursued in v1
- Censored-demand modeling — v2 only, on company data

**Deliberately undecided:**
Chronos-2 fine-tuning. Asymmetric loss weights. Deployment host. Second-project topic.

---

## 3. Models compared

All five scored on identical folds through one harness:

1. **Seasonal-naive** — predict this week using the same week last year. The floor.
2. **Degree-day regression** — OLS on heating/cooling degree days plus calendar features.
3. **LightGBM** — gradient boosting on lag features, rolling means, weather, holidays.
4. **SARIMAX** — classical time series with exogenous weather variable.
5. **Chronos-2** — pretrained foundation model, zero-shot, CPU-capable.

Which model wins is an **output** of the project, not an input. Report what happens,
including if the simplest approach wins.

---

## 4. Architecture requirements

These are locked from the first line of code, because they are what make later changes
cheap.

**Standard table contract.** Every data loader returns exactly three columns:
`unique_id` (series name), `ds` (date), `y` (value). Nothing downstream ever references
a source-specific column name. Rename to `y` at the boundary.

**Adapter layer.** One loader function per data source. Swapping to company data means
writing a new loader, not touching the pipeline.

**Normalize per era, then concatenate — never the reverse.** The era tables are
separately maintained, so column names, date formats, and permit-type vocabularies may
differ across the 2019→2020 boundary. Each era gets its own normalizing function that
outputs the standard shape; concatenation happens after. When plotting the weekly series,
inspect the seam closely: a level shift landing exactly on a January 1 boundary is a data
artifact, not seasonality.

**Model-agnostic harness.** One function takes any model, runs the 12-fold backtest,
returns MAE / RMSE / MASE / coverage by horizon. Adding a model is an afternoon.

**Target-agnostic harness.** The target is a parameter, never hardcoded. Swapping count
for valuation is passing a different table in.

**Subtypes as rows.** Splitting by residential/commercial means additional `unique_id`
values in the same table, not new plumbing.

**Design for a data-poor v2.** Company data will start at ~50 weeks against v1's ~800.
Keep the feature set modest; avoid techniques that only work with a decade of history.

---

## 5. Build order

Boring parts first. Ambitious parts layered on once the boring parts work.

1. Pull data, clean, aggregate to weekly, plot
2. Seasonal-naive baseline + evaluation harness
3. Feature engineering, degree days joined
4. Degree-day regression
5. LightGBM
6. SARIMAX
7. Chronos-2 zero-shot
8. Prediction intervals + coverage check
9. Holdout, once
10. Write-up

---

## 6. Milestones

| Date | Milestone | Definition of done |
|---|---|---|
| Sep 15 | Data checkpoint | Dataset IDs found and recorded; HVAC filter vocabulary assessed; era seam inspected; weekly series plotted; dataset confirmed; naive baseline computed; target revised and logged; valuation field assessed |
| Sep 30 | Harness | Rolling-origin backtest runs and scores any model; seasonal-naive scored |
| Oct 20 | First model | Degree-day regression, honestly evaluated |
| **Nov 3** | **Minimum viable v1** | **LightGBM + baseline + evaluation. Shippable as-is.** |
| Nov 24 | Comparison arms | SARIMAX and Chronos-2 added |
| Dec 5 | Intervals | Prediction intervals with coverage checked |
| Dec 15 | v1 complete | Holdout run once, write-up drafted, repo tidy |

November 3 is the load-bearing date. Past it, everything is upside.

**Weekly checkpoint:** one line in the decision log every week — date, hours spent,
what was finished. This is the only early warning that the schedule is slipping.

---

## 7. Roadmap beyond v1

| Window | Focus |
|---|---|
| Jan 2027 | Deploy (Streamlit), start logging every prediction and eventual actual |
| Feb – Jun 2027 | Second project, in a different area of data science |
| Jul – Sep 2027 | Return to v1: harvest forward-test results, extensions, write-up |
| Oct – Nov 2027 | Grad applications (Spring 2028 entry) |
| Post-acceptance | v2 on OceanAirFlow operational data |

**The January deployment date is critical.** If v1 is not deployed and logging before
the second project begins, no forward-test results accrue and the strongest element of
the project is lost.

**Running in parallel, starting now:** weekly company job log — date, job count, job
type, ZIP, revenue, crew count. Five minutes every Friday. This is the only item on the
roadmap that cannot be accelerated later.

---

## 8. Tech stack

**Week one:** Python 3.12+, VS Code with Python and Jupyter extensions, `venv`,
`pandas`, `numpy`, `matplotlib`, `requests`, `jupyter`. Data as CSV/Parquet in a folder.
Git + public GitHub repo.

**Added when the project demands it:** `statsforecast` (seasonal-naive, AutoARIMA),
`mlforecast` + `lightgbm`, `utilsforecast` (evaluation), `hierarchicalforecast`
(stage 6), `chronos-forecasting`, `streamlit` (January), `holidays`.

**External data:** Open-Meteo for degree days (free, no key). Note: at prediction time
you have a weather *forecast*, not an observation — use Open-Meteo's Historical Forecast
API so training and deployment see the same kind of input. AHRI monthly shipments and
FRED rates as optional covariates.

**Not used:** Docker, Airflow, cloud infrastructure, MLflow, dbt.

---

## 9. Known limitations to state in the write-up

Name these before a reader finds them.

- Permits capture installs and replacements only, not service and repair calls
- Not every replacement gets a permit; the data undercounts by an unknown and possibly
  time-varying amount
- The record date is the issue date, which leads work start by an unknown lag
- The series is assembled from separately-maintained era tables; residual discontinuity
  at the 2019/2020 seam cannot be fully ruled out
- Coverage is City of LA only, not Ventura or Orange County
- Seasonal-naive does not degrade with horizon while real models do, so the gap narrows
  as horizon grows
- The 2020–present table is live, so row counts and the series end date depend on
  the pull date; results are reproducible only against a stated pull date, and the
  final week is always partial
