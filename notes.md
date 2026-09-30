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
  weeks. The harness now gives fold-scored seasonal-naive MAE by horizon
  (22.2 / 28.7 / 36.8 / 35.1 / 25.4 / 25.8), and as expected none of them
  equal 42.8. Open: keep 42.8 as an orienting figure in the write-up, or
  drop it. Never compare a fold-scored model against it.

- Seasonal-naive MAE by horizon humps at h3–h4 (36.8, 35.1) and drops back
  at h5–h6. Read as small-sample noise (each horizon averages only 12
  misses), not degradation — seasonal-naive is always exactly 52 weeks back
  at every horizon. Untested: which specific weeks drive the h3/h4 bump.

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

- State of `backtest.ipynb`: `folds` (list of 12 DataFrames) stacked with
  `pd.concat` into `all_folds` — 72 rows, index labels 794–843 with
  repeats (folds step 4, windows are 6, so neighbors share 2 weeks).
  `miss` column = `(baseline − y).abs()`. `MAE_horizon` = seasonal-naive
  MAE by horizon. `target_horizon` = `MAE_horizon * 0.85`. Headline bar at
  h4 is **29.8** — pass/fail is h4 only, the other five are reported.
- Seasonal-naive rebuilt from `train` alone, verified in a scratch cell:
  `train['y'].iloc[-52:-46]`. Fold 1 gives 229 at label 742 (794 − 52);
  fold 12 gave 248 at 786. Still to do: compare all 6 fold-1 values
  against fold 1's `baseline` column, not just the first.
- **Pick up at:** turn that slice into a small seasonal-naive function —
  takes `train`, returns 6 guesses.
- Then: wrap the loop into one function — in: a model, out: that model's
  MAE by horizon (1–6). Run seasonal-naive through it and confirm it gives
  back the exact same six `MAE_horizon` numbers.
- `train` is built in the loop but never used — seasonal-naive reaches
  into `used_set` directly. The wrap fixes this.
- The loop's mask uses `weekly_hvac_permits` while the answer key uses
  `used_set`. Works (origins sit before the holdout), but decide on one
  table when wrapping.
- `/ len(folds)` in `MAE_horizon` assumes every horizon group has the same
  number of rows. Find the direct way to average each group.
- `range(12)` is hardcoded in the loop. Decide whether to tie it to
  `origins` instead.
- Small fixes in backtest: swap cell `[3]` (`index.dtype`, stale check)
  for `.dtypes`. Change `used_set` from plain brackets to `.iloc[:-26]`.
  Delete the scratch cells from building fold 1 (`first_slice`,
  `baseline2`, the list-comprehension cell) and from 2026-09-29
  (`train_scratch`, the `.iloc['839']` cell, the old `mean` cells, the
  `.loc[0]` cell).
- Decision log, if not already done: seasonal-naive MAE by horizon, h4
  bar = 29.8, with pull date.
- Once backtest has the origins, delete the origins cell in explore.
  One decision, one place.
- Harness milestone due Sep 30. Remaining: wrap only.
- Still outstanding: walk through the fetch code in `explore.ipynb`.
- Practice notebooks moved to `python-data-exploration` repo;
  `pull.rebase` left unset here, `--no-rebase` used per-pull.

## Next review

Carried over from the 2026-09-29 review and session. Covered but still
shaky. Bring them up when a review session is asked for.

- `.iloc[0]` vs `.index[0]` — `origins.iloc[0]` gives the value (the
  date, 2025-03-23). `origins.index[0]` gives the label (793). The mask
  needs the date; the answer-key slice needs the number to do +1, +7.
  Both `.loc` and `.iloc` return values — they differ in how they look
  it up (label vs position), not in what they return.
- Broadcasting vs vectorization vs index alignment — three different
  things. Broadcasting: one value to every row (`fold = 1`).
  Vectorization: one operation on every row of a column at once, no row
  loop (`ds <= origin`, `baseline − y`) — not "every column."
  Index alignment: assigning a Series matches by label, so mismatched
  labels give silent NaN.
- Counting dates — 52 weeks is 364 days, so "52 weeks back" lands one
  day off the calendar date. Every row is a Sunday; a non-Sunday date
  is always wrong. Count answer-key weeks from the origin (origin + 42
  days = last week).
- `len()` on a list vs on an item — `len(folds)` counts boxes (12).
  `len(folds[0])` counts rows in one box (6). `append` adds a whole
  table as one item; it doesn't unpack rows.
- Calling vs naming a function — `pd.concat` with no parentheses hands
  you the function itself, it doesn't run it. Assigning that to `folds`
  overwrote the list. Save results under a new name so inputs survive.
- Stale output — editing a cell doesn't rerun it. Same execution number,
  or a traceback line that doesn't match the cell's code, means the
  output is old.
- Kernel memory — variables stick around after a loop. `train` after the
  loop is fold 12's, not fold 1's.
- Indexing errors — `.iloc['839']` fails: `.iloc` needs an integer
  position, and 839 is a label (it also appears twice in `all_folds`).
  `.loc[0]` on a DataFrame returns one row, so `train['y']` became a
  single number and `.iloc` on it threw AttributeError.
- Negative positions — `[0]` is the first row, `[-1]` the last. The origin
  is `[-1]` in `train`. h1's guess is 51 rows before the origin → `-52`;
  h6's is 46 before → `-47`. Slices stop before the end → `-52:-46`.
- Seasonal-naive doesn't degrade with horizon — the gap between guess and
  target is always 52 weeks. Real models do degrade, so the model-vs-
  baseline gap narrows as horizon grows (charter §9).
- MASE bar — multiply, don't guess-and-divide: 0.85 × 35.08 = 29.8.
  (30 fails: 30 / 35.08 = 0.855.) MASE is a ratio, not a percent; under
  1 means the model beat the baseline.
- The wrap — pass in the **model**, not its guesses, so every model takes
  the identical test. Seasonal-naive is a student too, run through the
  same function; its MAE is the denominator for every MASE, forever.
