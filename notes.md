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

- **Harness milestone hit 2026-09-30, on the due date.** Next milestone:
  first model (degree-day regression) by Oct 20.
- State of `backtest.ipynb`: `seasonal_naive(train)` takes the full
  fold `train` table, slices `['y'].iloc[-52:-46]`, returns 6 guesses as a
  plain list (`.to_list()` strips labels, so no NaN from mismatched index).
  `score(model)` runs all 12 folds, calls `model(train)` for the guesses,
  stacks with `pd.concat`, builds `miss`, returns MAE by horizon.
  `score(seasonal_naive)` reproduces 22.2 / 28.7 / 36.8 / 35.1 / 25.4 /
  25.8 exactly — regression test passed. Fold 1's 6 guesses also checked
  against the old `baseline` column: all 6 match.
- **Pick up at:** `/ len(folds)` in the `MAE_horizon` line. Question left
  unanswered: what are you actually trying to get for each horizon group?
  One word. Then find the direct way to get it.
- The guesses column inside `score()` is still named `baseline`, but it
  now holds *whatever model's* guesses. Misleading once LightGBM runs
  through. Rename it. Same for `akey_guesses` (it's guesses, not answer
  key) and `train_scratch` inside `seasonal_naive` (cosmetic).
- `score()` returns MAE only. Charter §4 says the harness returns
  MAE / RMSE / MASE / coverage by horizon. Coverage waits for intervals
  (build step 8). Decide when to add RMSE and MASE. MASE denominator is
  always `score(seasonal_naive)`.
- The loop's mask uses `weekly_hvac_permits` while the answer key uses
  `used_set`. Works (origins sit before the holdout), but decide on one
  table.
- `range(12)` is hardcoded in the loop. Decide whether to tie it to
  `origins` instead.
- Small fixes in backtest: swap cell `[3]` (`index.dtype`, stale check)
  for `.dtypes`. Change `used_set` from plain brackets to `.iloc[:-26]`.
  Delete scratch cells: from building fold 1 (`first_slice`, `baseline2`,
  the list-comprehension cell); from 2026-09-29 (`train_scratch`, the
  `.iloc['839']` cell, the old `mean` cells, the `.loc[0]` cell); from
  2026-09-30 (`all_folds.iloc[794:800]`, the `.iloc[0:6]` check, the first
  `seasonal_naive(model)` version). The old standalone loop, concat,
  `miss`, and `MAE_horizon` cells now live inside `score()` — delete the
  standalone copies once you trust the function.
- Once backtest has the origins, delete the origins cell in explore.
  One decision, one place.
- Still outstanding: walk through the fetch code in `explore.ipynb`.
- Practice notebooks moved to `python-data-exploration` repo;
  `pull.rebase` left unset here, `--no-rebase` used per-pull.

## Next review

Carried over from the 2026-09-29 and 2026-09-30 sessions, trimmed by the
2026-10-06 review. Only items not yet solid. Each example describes its
table so it can be reviewed with only this file open.

- Index alignment — when you assign a Series into a DataFrame column,
  pandas matches rows by **label**, not by order. Matching labels get
  the value; no match gets NaN. No error is raised.
  Example: `all_folds` is 72 rows, labels 794–843 (some repeat).
  `all_folds.groupby('horizon')['miss'].mean()` is 6 rows, labels 1–6.
  Assigning it into `all_folds['mean']` matches zero labels → whole
  column NaN, silently. Store results like that in their own variable.

- Counting dates — every row is a Sunday (weeks labeled by Sunday end).
  Target week = origin + 7 × horizon days. Origin 2025-03-23, h6 →
  +42 days → 2025-05-04. A non-Sunday date is always wrong (2025-05-05
  is a Monday). "52 weeks back" = 364 days, not 365 (365 lands on a
  Saturday).

- Kernel memory — variables stay in the kernel until overwritten. The
  fold loop reassigns `train` every pass, so after the loop `train` is
  fold 12's. `train['y'].iloc[-52:-46]` then starts with 248 (fold 12).
  Rebuild `train` with `origins.iloc[0]` in the mask and the same slice
  starts with 229 (fold 1). Same slice, different data — the slice
  can't tell you which fold you're on; only `train` can.

- Indexing errors — `.iloc` = position (integers, 0 to len−1).
  `.loc` = label (what's printed on the left).
  Context: `all_folds` is 72 rows, positions 0–71, labels 794–843.
  - `all_folds.iloc['839']` → **TypeError** (wrong type of input):
    `.iloc` only takes integers. `.iloc[839]` would be **IndexError**
    (position out of range): only 72 positions exist.
  - `all_folds.iloc[794:800]` → comes back **empty, no error**. 794 is
    a label, not a position. A *slice* past the end gives nothing
    instead of erroring — a single position past the end errors.
  - `all_folds.loc[839]` → works. 839 is a label; it appears twice,
    so 2 rows come back.
  - `train = weekly_hvac_permits.loc[0]` then `train['y'].iloc[...]`
    → **AttributeError** (that thing has no such method): `.loc[0]`
    returns one row, so `train['y']` is one number, and a number has
    no `.iloc`.
  - Related: `used_set.iloc[origins.index[i] + 1 : ...]` feeds a label
    into `.iloc`. Works only because `used_set`'s labels are a 0-up
    counter (label 793 = position 793). If they stop matching, it
    silently grabs the wrong weeks.

- `.iloc[0:6]` to grab fold 1 worked only because fold 1 was stacked
  first. That's position luck. To get "rows where fold is 1," filter by
  the `fold` column, same move as the `mask` cell.

- MAE and MASE — answer from scratch next time.
  - MAE: each week's miss = |guess − actual|. MAE = average of those
    misses, in permits. Lower = better.
  - MASE: model MAE ÷ seasonal-naive MAE, same horizon, same folds.
    A ratio, not a percent. Under 1 = beat the baseline; over 1 = lost.
    1.10 = misses 10% bigger = 10% worse.
  - Project rule: MASE ≤ 0.85 at h4. Seasonal-naive h4 MAE = 35.08.
    Bar = 0.85 × 35.08 = 29.8 — the **highest** MAE that passes
    (ceiling). MAE 30 fails: 30 / 35.08 = 0.855 > 0.85. Multiply to
    get the bar; don't guess-and-divide.
  - Practice: a model scores MAE 28 at h4. What's its MASE? Pass?
    Another scores MASE 1.20 — better or worse than baseline, by how much?

- **Interface** — every model follows the same rule: `train` goes in, 6
  guesses come out. That's what lets one harness score any model.

- **Higher-order function** — a function that takes another function as
  an argument. `score(seasonal_naive)`: `score` doesn't care which model
  it gets, it just calls whatever shows up. Pass in the **model**, not
  its guesses, so every model takes the identical test.

- **Model vs data** — inside `score`, `model` is a function, so it's the
  thing you *call*: `model(train)`. `train` is data, so it goes *inside*
  the parentheses. Mistakes made: `model['ds']` (indexing a function
  like a table) and `seasonal_naive(model)` (flipped). Clue: VS Code
  grays out a variable that never gets used — gray `train` meant the
  guesses weren't coming from it.

- **Regression test** — rebuild something, confirm it gives the old
  answer. `score(seasonal_naive)` matching the six hand-built MAE numbers
  proves the wrap is correct before trusting it with a new model.

- The wrap's output is MAE at **all six horizons** (1–6), not just h4
  MASE. Charter requires every horizon reported. MASE is a separate
  step after.

- Seasonal-naive is a **student**, not the teacher. It runs through the
  same `score()` as every model. Its MAE is the bottom half of every
  MASE, forever — no baseline, no MASE.
