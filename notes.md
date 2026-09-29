# Working Notes

## What this file is for

A scratch file for open questions, unresolved oddities, and the next thing to do.
It exists so a new chat session can pick up where the last one left off without
re-deriving context.

**This is not the decision log.** The decision log is append-only and holds
*resolved* decisions with dates and reasons. This file holds the opposite:
things still in the air.

## How to maintain it

- **Delete items once they're resolved.** This file is meant to be edited and
  pruned. An item that has been settled does not belong here — it belongs in
  `decision-log.md` as a dated entry, and it should be removed from this file
  in the same sitting.
- If it never turns into a decision (just a fixed bug or a dead end), delete it
  without logging it anywhere.
- Keep it short. If this file is getting long, items are being added and not
  cleared.
- **Claude: at the end of a session, check this file and prompt me to remove
  anything that got resolved during it, and to add anything new that's still
  open.** Don't let it drift out of date.
- For this file to be visible to Claude, it must be added to the Claude project
  files, not just committed to the repo. Those are separate places.

---

## Open questions

- `CD` nulls: 113 in `5m3t-xjex`, 335 in `67is-svtd`. `ZIP_code` nulls: 13
  and 2. Negligible against 240k rows and both columns are stage-6 only, so
  not a problem now. Decide how to handle before any geographic split.

- COVID structural break, late March 2020. The series drops hard and takes
  time to recover. Not a seam artifact — confirmed by plot. Open: whether
  training folds spanning the break need any handling, or whether it's far
  enough back to leave alone. Note: excluding COVID weeks from *evaluation*
  is not an option — dropping weeks the baseline handled badly lowers the
  bar the model has to clear. Baseline and models score on identical weeks.
  Any handling would be on the training side only.

- The 42.8 baseline MAE is a whole-series number, computed once over 793
  weeks. Charter §2.6 defines MASE over identical evaluation windows in the
  rolling-origin backtest, so the harness will compute a seasonal-naive MAE
  per fold, and those will not equal 42.8. Open: whether 42.8 is kept as an
  orienting figure or dropped once the harness exists. Do not compare a
  fold-scored model against it.

- Series length moves with the pull date — 870 on 2026-09-10, 871 on
  2026-09-15, 872 on 2026-09-21, since `67is-svtd` is live. Any recorded
  count needs a pull date attached. Origins are pinned by position so this
  no longer moves the folds, but it does move `used_set`'s length. The saved
  parquet is a snapshot: re-export it from `explore.ipynb` after every pull,
  or `backtest.ipynb` reads stale data.

- Pinned origins mean the backtest stops seeing new data as the series
  grows. Accepted deliberately for comparability across model runs. Open:
  whether to re-pin once before the December write-up so the reported
  numbers aren't scored on a window that's months stale by then.

- Seasonal amplitude may vary by year. 2010–2013 were flat — in 2010 the
  peak beat the runner-up by ~16 permits in a column near 260 — while later
  years swing harder. Only those four were examined closely, so "the pattern
  is clearer from 2014 on" is untested. To settle it: compute the 23%
  amplitude measure per year instead of once across the whole series, and
  see whether it trends. Matters because if old years carry a weaker
  seasonal signal, they may be worth less as training data.

- Summer 2026 is the highest stretch in the whole series. Not explained yet.

- 2025 sits low across the whole column — 187 in January, most months
  230–270, against 2018 in the 300s. Level question, not seasonality.
  Sits alongside the unexplained summer 2026 high. Related: collapsing the
  pivot down the year axis (`mean(axis=0)`, 2026 excluded) gives a 27%
  spread across years, high 2018, low 2020. Year-to-year level moves more
  than the seasonal swing does — worth keeping in mind before assuming
  seasonality is the dominant structure.

- Year-level drift (27%) exceeds seasonal amplitude (23%), and seasonal-naive
  cannot track level — it copies last year's level wholesale. This is the
  basis for suspecting the 0.85 target is soft. Untested. Worth checking once
  the first real model is scored: how much of the baseline's error is level
  error rather than seasonal error.

## Next session

- Fold loop in `backtest.ipynb` runs clean, no warnings. `folds` is a
  list of 12 DataFrames, 6 rows each. Columns: ds, y, unique_id,
  baseline, fold (1–12), horizon (1–6). `folds[-1]` checked: 838–843,
  2026-02-01 to 2026-03-08, fold 12, horizons 1–6.
- Pick up at: verify `folds[-1]` baseline. First row is 248; confirm it
  equals `used_set['y']` at position 786 (838 − 52). One line.
- Then: stack the 12 into one 72-row table. Find the pandas function
  that stacks a list of DataFrames. Predict row count and index labels
  before running.
- Then: score it. MAE of seasonal-naive by horizon (1–6).
- Then: wrap it so any model can plug in, not just seasonal-naive
  (charter §4, model-agnostic harness).
- `range(12)` is hardcoded in the loop. Decide whether to tie it to
  `origins` instead.
- Small fixes in backtest: swap cell `[3]` (`index.dtype`, stale check)
  for `.dtypes`. Change `used_set` from plain brackets to `.iloc[:-26]`.
  Delete the scratch cells from building fold 1 (`first_slice`,
  `baseline2`, the list-comprehension cell).
- Once backtest has the origins, delete the origins cell in explore.
  One decision, one place.
- Harness milestone due Sep 30. Remaining: stack, score, wrap.
- Still outstanding: walk through the fetch code in `explore.ipynb`.
- Practice notebooks moved to `python-data-exploration` repo;
  `pull.rebase` left unset here, `--no-rebase` used per-pull.

## Next review

Carried over from the 2026-09-29 review. These were covered but still
shaky. Bring them up when a review session is asked for.

- `.iloc[0]` vs `.index[0]` — `origins.iloc[0]` gives the value (the
  date, 2025-03-23). `origins.index[0]` gives the label (793). The mask
  needs the date; the answer-key slice needs the number to do +1, +7.
  Both `.loc` and `.iloc` return values — they differ in how they look
  it up (label vs position), not in what they return.
- Broadcasting vs vectorization vs index alignment — three different
  things. Broadcasting: one value to every row (`fold = 1`).
  Vectorization: an operation on a whole column at once, no row loop
  (`ds <= origin`). Index alignment: assigning a Series matches by
  label, so mismatched labels give silent NaN.
- Counting dates — 52 weeks is 364 days, so "52 weeks back" lands one
  day off the calendar date. Every row is a Sunday; a non-Sunday date
  is always wrong. Count answer-key weeks from the origin (origin + 42
  days = last week).
- `len()` on a list vs on an item — `len(folds)` counts boxes (12).
  `len(folds[0])` counts rows in one box (6). `append` adds a whole
  table as one item; it doesn't unpack rows.
