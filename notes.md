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

- Climatology is the only weather input for the forecast weeks, so the
  weather feature is a smooth "typical year" curve. Seasonal-naive already
  copies last year's seasonal shape. Untested: how much degree days add on
  top of that. If degree-day regression barely beats the baseline, that's a
  real result to report, not a failure to fix.

- Weather archive (ERA5-Land) runs about a week behind today — last 7 days
  came back NaN on the 2026-10-07 pull (unpublished, not missing). Doesn't
  matter now (permit parquet ends 2026-09-20). Will matter in January when
  deployed and pulling fresh. Decide then.

## Next session

- **First model (degree-day regression) due Oct 20.** Harness done Sep 30.
- **OWED: decision log entries from 2026-10-07 (weather).** Write these
  first thing:
  1. Weather input for all 6 forecast weeks = climatology. Deviates from
     charter §8. Reason: Open-Meteo Historical Forecast API stitches the
     first few hours of each run (close to observed weather, not multi-week
     forecasts) and only covers ~2021–22 onward. Searched alternatives:
     NOAA CPC week 3–4 outlooks (above/below-normal odds only, official
     for temperature from May 2017) and Open-Meteo Previous Runs API (1–7
     day lead times, from 2024). No usable 3–6 week forecast archive back
     to 2010. Methodological reason, not score-based.
  2. Climatology built only from years before each fold's origin
     (anything later is leakage).
  3. Three weather spots, one per zone, top permit ZIP in each:
     90026 Central (34.07883, -118.2637), 90045 Westside (33.95302,
     -118.40028), 91367 Valley (34.174899, -118.615271). Reason: permits
     spread across the city (top ZIP ~2%, top 15 ~25%), no single spot
     represents it. Degree days per spot per day, then equal-weighted
     average. Equal because zone shares are similar, and permit-count
     weights would use all years (leakage).
  4. Historical Weather API, `models=era5_land`. One model for the whole
     range avoids the 2017 model switch (era-seam-style artifact). 9 km
     grid keeps the three spots in separate squares; ERA5's ~25 km grid
     could merge them.
  5. Degree-day definition: base 65°F, daily temp = (max + min) / 2,
     Fahrenheit, America/Los_Angeles timezone.
  6. Weather gets its own notebook, `weather.ipynb` — charter §4, one
     loader per source. `explore.ipynb` not renamed (decision log
     references it).
- State of `backtest.ipynb` (clean Restart + Run All passed 2026-10-07):
  `score(model)` loops `range(len(origins))`, cuts both `train` and `akey`
  from `used_set`, stores guesses as `predicted` → column `prediction`,
  MAE via `groupby('horizon')['miss'].mean()`. `baseline_MAE =
  score(seasonal_naive)`, `target_horizon = baseline_MAE * .85`
  (h4 bar 29.8). Scratch cells deleted.
- State of `weather.ipynb`: URL locked:
  `https://archive-api.open-meteo.com/v1/archive?latitude=34.07883,33.95302,34.174899&longitude=-118.2637,-118.40028,-118.615271&start_date=2010-01-03&end_date=2026-10-07&daily=temperature_2m_min,temperature_2m_max&models=era5_land&timezone=America%2FLos_Angeles&temperature_unit=fahrenheit`
  `data = resp.json()` is a list of 3 dicts, same order as the URL
  (Central, Westside, Valley). `first_weather_df` built from
  `data[0]['daily']`: 6122 rows, 7 NaN per temp column, all at the end
  (Oct 1–7, unpublished). No holes in the middle.
- **Pick up at:** loop over the 3 spots → DataFrame from each `'daily'`,
  add a zone column, append to a list, `pd.concat` once after. Open
  question: how does each pass know its zone name? Call the row count
  before running.
- After that (one at a time): daily temp → CDD/HDD per spot per day →
  equal-weight average across spots per day → weekly sum (W-SUN) → save
  parquet. Climatology gets built per fold later, from pre-origin years.
- `score()` returns MAE only. Charter §4 says the harness returns
  MAE / RMSE / MASE / coverage by horizon. Coverage waits for intervals
  (build step 8). Decide when to add RMSE and MASE. MASE denominator is
  always `baseline_MAE`.
- Small leftovers in backtest: confirm cell `[3]` (`index.dtype`) was
  swapped for `.dtypes`. `train_scratch` name inside `seasonal_naive`
  (cosmetic).
- Once backtest has the origins, delete the origins cell in explore.
  One decision, one place.
- Still outstanding: walk through the fetch code in `explore.ipynb`.
- Practice notebooks moved to `python-data-exploration` repo;
  `pull.rebase` left unset here, `--no-rebase` used per-pull.

## Next review

Carried over from earlier sessions plus everything flagged 2026-10-06 and
2026-10-07. Review at the end of the project chat. Each example describes
its table so it can be reviewed with only this file open.

### Carried over

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
  Fix: Restart kernel → Run All. Execution counts restart at [1] and go
  in order. If they don't, you didn't restart.

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

- Filtering rows by a column value — to get fold 1 out of `all_folds`
  (72 rows; columns `ds`, `y`, `unique_id`, `prediction`, `fold`,
  `horizon`, `miss`), ask for what you mean: "rows where `fold` is 1."
  1. Compare a column to a value → a True/False Series, one per row.
  2. Hand that True/False Series to `.loc[ ]` → keeps only the True rows.
  Why not `.iloc[0:6]`: that means "first 6 rows." It matched fold 1
  only because fold 1 was appended first. Reorder `all_folds` and it
  silently grabs the wrong rows.

- MAE and MASE — answer from scratch next time.
  - MAE = **Mean** Absolute Error. Each week's miss = |guess − actual|.
    MAE = the mean (average) of those misses, in permits. Lower = better.
    (On 2026-10-07 said M = "miss." Wrong. M = Mean.)
  - MASE: model MAE ÷ seasonal-naive MAE, same horizon, same folds.
    Under 1 = beat the baseline; over 1 = lost. 1.10 = 10% worse.
  - Project rule: MASE ≤ 0.85 at h4. Bar = 0.85 × 35.08 = 29.8, the
    **highest** MAE that passes.
  - Practice: a model scores MAE 28 at h4. What's its MASE? Pass?
    Another scores MASE 1.20 — better or worse than baseline, by how much?

- **Two functions, not one** — `seasonal_naive(train)` returns **6
  guesses**. `score(model)` runs all 12 folds and returns **6 MAE
  numbers**. Baseline MAE comes from `score(seasonal_naive)`, never from
  `seasonal_naive` alone. (Made this mistake again 2026-10-07 when
  rebuilding `target_horizon`.)

- **Interface** — only what goes in and what comes out. Harness plug:
  **in** = `train`, **out** = 6 guesses as a plain list. The model never
  knows which fold it's on; `score` picks the origin and cuts `train`.

- **Regression test** — change something, then check it still gives the
  **old, known answer**. **Call the numbers out loud before you run it** —
  that's what makes it a test instead of just looking. On 2026-10-06 it
  caught a real bug: `range(len(origins) - 1)` ran 11 folds; decimals like
  .181818 (elevenths) gave it away.

- **Data leakage** — not yet answered. Explain first next time.
  Context: the old loop made seasonal-naive guesses by reaching into
  `used_set`, which includes weeks after each fold's origin. The new
  `seasonal_naive` only gets `train`, which stops at the origin.
  Question: what is data leakage, and why does it matter that the model
  only gets `train`? (Hint: see train/serve mismatch and climatology
  below — both are leakage cases.)

- Harness output is MAE at **all six horizons**, not just h4 MASE.

- Seasonal-naive is a **student**, not the teacher. Same `score()` as
  every model. Its MAE is the bottom half of every MASE, forever.

### pandas and Python (2026-10-06 / 10-07)

- `.size()` vs `.count()` vs `.sum()` — all work after `groupby`.
  - `.size()` — how many **rows** in each group. Counts everything,
    NaN included.
  - `.count()` — how many **non-NaN values** in each group. Same as
    `.size()` unless there are NaNs.
  - `.sum()` — **adds up** the values. Only makes sense on numbers you
    want totaled.
  Example: `whole_f.groupby('ZIP_code')['permit_nbr'].size()` = permits
  per ZIP. `.sum()` there would add permit *numbers* together — garbage.
- `.mean()` — the average, directly. Replaced
  `.sum() / len(folds)` in `score()`. The hand-built version only works
  if every group has exactly 12 rows; `.mean()` counts each group itself.
- `.sort_values(ascending=False)` — sorts biggest first. Default is
  `ascending=True` (smallest first).
- **Arguments / parameters** — the extra settings you pass in the
  parentheses, like `ascending=False`, `ignore_index=True`. Many have a
  default, so you only pass them to change the default.
- `.isna()` — True where a value is missing. `.isna().sum()` counts the
  missing values, because Python counts **True as 1, False as 0**.
- `range(len(x))` — already hits every item in `x`, 0 to len−1. No `- 1`.
  The "last one excluded" is what makes zero-indexing work out.
- `type()` — tells you what kind of thing something is (`list`, `dict`,
  `Response`...). Use it before you try to work with something unfamiliar.
- Negative indexing — `.iloc[:-26]` = from the start up to, not
  including, the last 26 rows.
- Reading a traceback — **read the last line first.** Everything above it
  is usually library internals. `KeyError: 'baseline'` = you asked for
  something by name that isn't there (the column had been renamed).
  Loud errors are the good kind; silent ones (NaN columns) are worse.
- **Check which table a result came from** before reading it. ZIP counts
  on `data_frame` (2010–2019 only) gave a different #1 than `whole_f`
  (both eras).

### APIs and data sources

- `resp` vs `resp.json()` — `resp` is the **envelope** (status code,
  headers). `resp.json()` **opens it** into Python lists and dicts. Save
  it once: `data = resp.json()`.
- **Status codes** — 200 = OK. 400s = you asked wrong. 500s = their side
  broke.
- **Check every API setting matches what you're building** — units,
  timezone, coordinates, model. Default Open-Meteo is Celsius: a 30°C
  (86°F) day through a 65 base gives CDD 0, HDD 35 — a hot day shows up
  as cold, with no error.
- **Misalignment** — data that's supposed to line up doesn't (e.g. GMT
  "days" run ~5pm to 5pm LA time, splitting hot afternoons across two
  days). Not the same as leakage.
- **Reanalysis** — past weather rebuilt by feeding old real measurements
  into a weather model. Complete and gap-free, but built after the fact,
  so the newest ~week isn't published yet.
- **Grid squares** — the model chops the map into squares; every spot in
  a square gets the same value. ERA5 ~25 km (spots 15–35 km apart could
  merge), ERA5-Land ~9 km (each spot keeps its own weather). Bigger
  squares = blurrier, like big pixels. Our pick: ERA5-Land. You asked for
  34.07883 and got 34.1 back — that's the grid snapping.
- **NaN at the end vs. the middle** — NaN at the very end of a live source
  usually means "not published yet." NaN in the middle is a real hole.

### Concepts (2026-10-07)

- **Holdout contamination** — the 26 holdout weeks getting touched at all
  before December (training, scoring, peeking). Cousin of data leakage:
  leakage is a model seeing the future inside a fold; contamination is the
  final-test weeks touched during development. Fix: `train` and `akey` in
  `score()` both slice from `used_set`.
- **Degree days** — base 65°F. CDD = how far the day's temp is above 65.
  HDD = how far below. The other one is 0, never negative.
  80°F day → CDD 15, HDD 0.
- **Averaging hides extremes** — week of 3 days at 50°F + 4 at 80°F:
  average ~67°F looks mild (≈2 CDD, 0 HDD), but real totals are 60 CDD
  and 45 HDD.
- **Rule:** when a quantity can't go negative per day, compute it per day
  before you combine. Applies across days and across locations.
- **Train/serve mismatch** — what the model sees in training must match
  what it sees when it runs for real. Training on actual future weather
  when it'll only have forecasts in January = scores look better than
  reality. A form of leakage.
- **Climatology** (climate normal) — "what this week is usually like":
  average the same week's degree days across years. Steadier than one
  year back (no freak cold snap). Only use years **before** the fold's
  origin.
- **Methodological reason** — the only kind of reason allowed to change
  the backtest or a locked charter choice: something about the method or
  data is wrong. Never "a different design scored better."
- **Data artifact** — a pattern created by how the data was collected or
  stitched, not by the real world.
- **Era seam** — where two data sources are stitched together (permits:
  2019→2020 tables; weather: 2017 model switch). A jump landing exactly on
  the seam is an artifact. Fix for weather: one model (ERA5-Land) for the
  whole range.
- **Parquet** — file format that saves a table *with* its types (dates
  stay dates). The handoff between notebooks.
- **One notebook, one job** (any project) — `explore.ipynb` checks permit
  data, `weather.ipynb` pulls and builds weather, `backtest.ipynb` is the
  judge. Each saves a parquet for the next. Don't make the judge do two
  jobs.
- **LA zones** —
  - **Valley:** San Fernando Valley, north of the hills. Inland, no ocean
    breeze, hottest summers. (91367, 91331, 91335, 91342)
  - **Westside:** west of downtown toward Santa Monica and the beach.
    Ocean breeze, mildest. (90045, 90025, 90049, 90024, 90066)
  - **Central:** downtown, Hollywood, Mid-City. In between.
    (90026, 90019, 90036, 90016, 90042, 90027)
