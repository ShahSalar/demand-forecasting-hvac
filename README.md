# demand-forecasting-hvac

Forecasting weekly HVAC permit counts for the City of Los Angeles, six weeks ahead.

This is a learning project. The goal is a working forecast backed by an honest evaluation, with the reasoning behind each decision written down as I go.

## Data

Permit records published by the Los Angeles Department of Building and Safety (LADBS), pulled from the Socrata API at `data.lacity.org`.

Two era tables are combined into a single frame:

Dataset ID  | Coverage     | Notes
--------------------------------------------------------
`5m3t-xjex` | 2010–2019    | Static
`67is-svtd` | 2020–present | Live, updates continuously

Both tables share an identical 31-column schema, so no normalization is needed at the 2019/2020 seam. Records are filtered on `permit_type = 'HVAC'` and rolled up to a weekly count.

As of the 2026-09-17 pull: 870 complete weeks, averaging roughly 276 permits per week.

## Goal

Forecast the weekly permit count six weeks ahead and beat a seasonal-naive baseline by a meaningful margin.

- Baseline: seasonal naive, which predicts each week using the same week one year earlier.
- Metric: MASE, targeting ≤ 0.85 against that baseline.

The target is only meaningful if the series has genuine repeating seasonality. Confirming that is part of the validation work below, not an assumption.

## Status

**Data validation in progress. No model built yet.**

Three checks have to pass before any modeling starts:

Check       | Status      | Finding
----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
Gaps        | Complete    | 15 lowest weeks are all explained by COVID lockdowns (March–April 2020) or holiday shortened weeks (Thanksgiving, Christmas, New Year). No unexplained gaps.
Volume      | Complete    | Mean of ~276 permits/week across 870 weeks. Enough signal relative to week to week noise.
Seasonality | In progress | Building a month by year table of average weekly counts.

Next up after that: a seasonal naive baseline, then backtest fold design, then models.

## Repository layout

```
charter.md         Project scope, milestones, gate checks, known limitations
decision-log.md    Append only record of decisions with the numbers behind them
notes.md           Working notes and open questions
notebooks/         Jupyter notebooks; explore.ipynb is the active one
data/              Gitignored — raw pulls are not committed
```

Notebook outputs are committed on purpose. Because the upstream data is live, those outputs are the only persistent record of what the raw data looked like at a given pull date.

## Setup

Python 3.12. Packages are installed with pip for consistency with the modeling dependencies added later.

```bash
conda create -n hvac python=3.12
conda activate hvac
pip install -r requirements.txt
jupyter lab
```

## Limitations

**The recent data is live.** `67is-svtd` updates continuously, so row counts and the series end date shift with every pull. Results are only reproducible against a stated pull date. Two consequences:

1. Running the same code on two different days produces different predictions.
2. The final week of any pull is always partial. If the model is evaluated against that truncated week it will look far worse than it is, because the permits for the last day or two simply haven't been filed yet. The partial week is structural, not missing data.

**There is a structural break in the series.** Permit volume drops sharply in late March 2020 and the post-2020 level does not return to the pre-2020 baseline. This has implications for how backtest folds are designed and for whether pre-2020 history should be weighted the same as recent history. Logged as an open item.

**Socrata paginates silently.** The API applies a default 1,000 row limit unless it's overridden. Any pull that doesn't set the limit explicitly will look complete and be wrong.
