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
  2026-09-15, 872 on 2026-09-21, 876 on <pull date> (last week
  2026-10-18), since `67is-svtd` is live. Any recorded count needs a pull
  date attached. Origins are pinned by position so this no longer moves
  the folds, but it does move `used_set`'s length. The saved parquet is a
  snapshot: re-export it from `explore.ipynb` after every pull, or
  `backtest.ipynb` reads stale data.

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

- Week 53 climatology is thin. ISO week 53 only exists in a few years
  (2015 and 2020 inside the backtest), so its average comes from about 2
  rows. No backtest fold forecasts a week 53. Could matter at deployment:
  2026 has a week 53 (late December). Decide then.

- OLS tooling. Charter §8 doesn't list statsmodels or scikit-learn. Options:
  do it with `numpy` (already installed), or add a package. Adding one
  deviates from §8, so log it. Decide at the start of the regression fit.

- `train['week'] = ...` adds a column to the `train` slice. No warning in
  the rehearsal. Inside the model function it might throw one. If it does,
  read it before fixing.

- Weather archive (ERA5-Land) runs about a week behind today. On the
  2026-10-07 pull the last 7 days came back NaN (unpublished, not missing).
  On the 2026-10-23 pull they came back filled, 0 NaN anywhere, because the
  URL's end date (Oct 7) was already past the ~7-day lag. Rule: compare the
  requested end date against "today minus the lag." Doesn't matter now.
  Will matter in January when deployed and pulling fresh. Decide then.

- Weather data depends on the pull date, same as permits. Current
  `sums_of_degree_days.parquet` was built from the 2026-10-23 pull and ends
  2026-10-04. After the 2026-10-28 join, the last 2 permit weeks
  (2026-10-11, 2026-10-18) have NaN CDD/HDD. Both sit in the 26-week
  holdout, so no fold reads them. Re-pull weather and re-save before the
  holdout runs. If the URL or end date changes, re-run `weather.ipynb` and
  re-save.

- Hot-day CDD spot check was skipped on 2026-10-23. Only a cold day (row 0)
  was checked by hand, and the "CDD > 0 and HDD > 0" check can't catch a
  flipped sign. Untested: pull the highest `daily_avg` row and confirm
  CDD = avg − 65, HDD = 0.

## Next session

- **Timeline (2026-10-28):** Order decided: degree-day regression first,
  cutoff **Oct 31**. If it isn't scored by then, park it and go straight to
  LightGBM. **Nov 3 MVP is the load-bearing date** (LightGBM + baseline +
  evaluation), 6 days out. LightGBM not started. Weather join and
  climatology are shared plumbing, so they count toward LightGBM too.
- Weather decisions logged 2026-10-07 (6 entries). Fix flagged typos if
  not done.
- State of `backtest.ipynb` (2026-10-28):
  - Weather loaded as `sum_of_degree_days` (874, 3).
  - `permits_and_weather = weekly_hvac_permits.merge(sum_of_degree_days,
    how='left')` → 876 rows × 5 columns (`ds`, `y`, `unique_id`, `CDD`,
    `HDD`). Last 2 rows NaN weather (inside holdout).
  - `used_set = permits_and_weather.iloc[:-26]` — one definition only,
    placed after the merge. `score()` unchanged; it reads `used_set`.
  - Regression test passed after Restart + Run All: 22.2 / 28.7 / 36.8 /
    35.1 / 25.4 / 25.8.
  - `baseline_MAE = score(seasonal_naive)`, `target_horizon =
    baseline_MAE * .85` (h4 bar 29.8).
- Rehearsal cells in `backtest.ipynb` (fold 1, scratch):
  - `train` = fold 1 cut from `permits_and_weather` with `mask` (794 rows).
  - `train['week'] = train['ds'].dt.isocalendar()['week']`
  - `climatology = train.groupby('week')[['CDD','HDD']].mean()` → (53, 2),
    `week` is the index.
  - `last_date = train['ds'].iloc[-1]` → 2025-03-23.
  - `forecast_weeks_fold_1` = list comprehension, `last_date +
    pd.Timedelta(weeks=i)` for i in 1–6, then turned into a Series.
    That cell overwrites its own input (rerun-unsafe).
  - `climatology_fold_1_predict` = the 6 week numbers (13–18). Name says
    climatology but it holds week numbers.
  - `climatology.loc[climatology_fold_1_predict]` → 6 rows CDD/HDD.
  - Loose global `train` exists now. Delete these scratch cells once the
    model function works, so it can't get mixed up with the `train`
    inside `score()`.
- **Pick up at:** fit the regression on fold 1's `train` (y on CDD and
  HDD). First decide OLS tooling (see Open questions). Then plug the 6
  climatology rows in to get 6 guesses as a plain list. Then move every
  step into `degree_day_regression(train)` next to `seasonal_naive`, run
  `score()`, compare h4 against 29.8. Calendar features per charter §3
  after that.
- `score()` returns MAE only. Charter §4 says the harness returns
  MAE / RMSE / MASE / coverage by horizon. Coverage waits for intervals
  (build step 8). Decide when to add RMSE and MASE. MASE denominator is
  always `baseline_MAE`.
- **Cleanup in `weather.ipynb`:** cells that overwrite their own input
  (`three_average_week = three_average_week.reset_index()`, and the
  rename cell) are rerun-unsafe — fold into one line writing a new name
  (e.g. `weekly_dd`). Names `three_average` / `taw` / `three_average_week`
  are vague. Delete scratch cells (failed concat attempts, `.size()` check,
  early test loops, `first_weather_df`). Then Restart + Run All and confirm
  (874, 3) again.
- State of `weather.ipynb` (2026-10-23): URL locked:
  `https://archive-api.open-meteo.com/v1/archive?latitude=34.07883,33.95302,34.174899&longitude=-118.2637,-118.40028,-118.615271&start_date=2010-01-03&end_date=2026-10-07&daily=temperature_2m_min,temperature_2m_max&models=era5_land&timezone=America%2FLos_Angeles&temperature_unit=fahrenheit`
  - `data = resp.json()` is a list of 3 dicts (Central, Westside, Valley,
    same order as the URL). Each `'daily'` is a dict of 3 lists, 6122 days.
  - Loop over `data` with `location = ['Central', 'Westside', 'Valley']`
    → DataFrame per spot → `zone` column → append to `all_zones` →
    `df_zones = pd.concat(all_zones)` → `reset_index(drop=True)`.
    18,366 rows. 0 NaN. `time` parsed with `pd.to_datetime`.
  - `daily_avg` = (min + max) / 2. `CDD` = (avg − 65).clip(lower=0).
    `HDD` = (65 − avg).clip(lower=0).
  - `three_average` = `groupby('time')[['CDD','HDD']].mean()` → 6122 days.
  - `taw` = resample `W-SUN` sum → 876 weeks → `three_average_week` =
    `taw.iloc[1:-1]` → 874 weeks → `reset_index()` + rename `time` → `ds`.
  - Saved `../data/sums_of_degree_days.parquet`. Read back verified:
    (874, 3), columns `ds` / `CDD` / `HDD`, `ds` is datetime64, first `ds`
    2010-01-10 (matches permits).
- Small leftovers in backtest: confirm cell `[3]` (`index.dtype`) was
  swapped for `.dtypes`. `train_scratch` name inside `seasonal_naive`
  (cosmetic).
- Once backtest has the origins, delete the origins cell in explore.
  One decision, one place.
- Still outstanding: walk through the fetch code in `explore.ipynb`.
- Practice notebooks moved to `python-data-exploration` repo;
  `pull.rebase` left unset here, `--no-rebase` used per-pull.

## Review style

How reviews run. Claude follows this every review session. Update it when
something isn't clicking.

- **One question at a time.** Show progress: *Topic X/N · Block Y*.
- **I answer first.** Claude doesn't hint before I try.
- **One clean question.** No second clause or parenthetical that gives
  away the answer to the first part. Ask it, then stop.
- **If I'm right:** one line confirming, then move on. No follow-up, no
  extra depth. Don't spend energy on things I already know.
- **If I'm wrong or half-right, explain in this order:**
  1. Say it's wrong in one line.
  2. **Tiny picture example:** a made-up table of 3–5 rows I can see,
     not the real 72-row table.
  3. **What happens,** shown on that tiny table.
  4. **One-line rule** to remember.
  5. One simple check question. If I get it, move on.
- **Ask the same question twice, max.** If I skip it twice or say move on,
  move on.
- **Plain words.** Short sentences. No new terms inside an explanation.
  Everyday comparison first if it helps (outlets, packages, pixels,
  fabric seams).
- **Give context in the question** when it needs code or numbers I might
  not have memorized: show the line, the table, or the setup. I won't
  have the code in front of me.
- **Don't stop the flow for naming** mid-build. Save naming notes for
  cleanup, unless the name causes a real bug.
- **Wrong or shaky topics** go on a comeback list, re-asked later with a
  fresh example. If I say "don't add it," don't.
- **Don't ask** date-counting by hand (I check those with code).
- If I've explicitly asked for a better explanation on a topic, give it
  a new angle, not the same words again.

## Next review

Comeback list from the 2026-10-22 review (42 topics done, these 9 were
wrong or shaky), plus items from the 2026-10-23 and 2026-10-28 build
sessions. Use the Review style above. Each has a fresh angle to try.

1. **Index alignment + why the model returns a plain list.** I asked for a
   better explanation. Fresh angle: pandas is like a mail carrier matching
   house numbers. A Series is a stack of letters with house numbers on
   them; a plain list is letters with no numbers, handed out in order.
   - Tiny table: 3 rows labeled 10, 11, 12. A Series labeled 0, 1, 2 →
     no house numbers match → all NaN, no error.
   - Same 3 values as a plain list → go in order, row 10, 11, 12.
   - Why it matters: `score()` drops the model's guesses into `akey`
     (labels like 794–799). A Series from the model would carry its own
     labels and come out NaN. A plain list can't misalign.
   - Also: 6 numbers can't sit in a 72-row table. Keep results like that
     in their own variable. (`reset_axis` doesn't exist; `reset_index`
     wouldn't fix it either.)
   - 2026-10-28 example of alignment working *for* me:
     `train['week'] = train['ds'].dt.isocalendar()['week']` lands on the
     right rows because the labels carry through every step.

2. **Kernel memory + function scope.**
   - A loop overwrites a variable every pass; after it ends, the variable
     holds the **last** pass (fold 12's `train`).
   - Variables made **inside a function** vanish when it ends. `train` now
     lives inside `score()`, so typing `train` in a new cell → NameError.
   - Fix for stale memory: Restart kernel → Run All; counts start at [1].
   - 2026-10-28: ran a regression test at `[62]`/`[63]` without
     restarting. The count is the tell.

3. **Regression test — the definition, not just the habit.**
   - Definition: change code, then check it still gives the **old, known
     answer**. "Regression" = going backwards (code breaking what used to
     work). Not the regression model.
   - Habit: call the numbers before running.
   - Example: rewrote `score()` with `.mean()`, checked it still gave
     22.2 / 28.7 / 36.8 / 35.1 / 25.4 / 25.8. Did it again 2026-10-28
     after adding weather columns to `used_set`.

4. **Lists vs. dicts, with examples (+ `type()`).** Asked to add again
   2026-10-23 — still shaky.
   - Like checking a box's label before opening it.
   - `list` → open with a **number**: `data[0]`.
   - `dict` → open with a **name (key)**: `data[0]['daily']`. No rows or
     columns — that's a table (DataFrame).
   - Wrong opener → error (lists don't take names; KeyError on a missing
     dict key).
   - Fresh angle from 2026-10-23:
     - `'daily'` is a **dict of lists**: 3 keys (`time`, min, max), each
       holding a list 6122 long. `len(daily)` → **3** (counts keys).
       `pd.DataFrame(daily)` → 3 columns, 6122 rows. Check: 5 keys × 100
       values → `len` 5, shape (100, 5).
     - `all_zones` is a **list of tables**, not a list of lists.
     - `pd.concat(all_zones)` ✅ vs `pd.concat([all_zones])` ❌ — box
       inside a box → TypeError.
     - `rename(columns={'time': 'ds'})` uses a dict: old name → new name.
     - When to pick which: **list** to keep things in order, **dict** to
       grab things by name.

5. **`resp` vs `resp.json()`.**
   - Amazon package: `resp` is the box + shipping label (`<Response
     [200]>`). `.json()` opens it and gives what's inside as Python
     lists/dicts.
   - Can't grab temperatures from `resp` directly. Save once:
     `data = resp.json()`.
   - Check question to use: `resp.json()[0]` — which spot? (Central,
     90026, first in the URL.)

6. **Grid squares.** Got backwards twice, clicked on the third try.
   - Every spot in one square gets the **same** temperature.
   - 25 km (ERA5) = big squares = **less** detail; Echo Park and
     Westchester could share one → identical temps.
   - 9 km (ERA5-Land) = small squares = **more** detail; each spot its own.
   - Pixels: big = blurry, small = sharp.

7. **Data artifact (vs. era seam).**
   - **Era seam** = a **place**: where two sources get glued (permits at
     Jan 2020; weather model switch 2017). Like the stitch line in fabric.
   - **Data artifact** = a **problem**: a fake pattern from how data was
     collected or stitched, not the real world. Like a color mismatch.
   - A seam can be clean (permits were). An artifact can happen without a
     seam (Celsius mistake → fake cold days).
   - COVID drop = real, not an artifact.

8. **Where models live.**
   - One notebook = one **job**, not one per new thing.
   - New **data source** = new job → new notebook (`weather.ipynb`).
   - New **model** = just a function that takes `train`, returns 6
     guesses → lives in `backtest.ipynb` next to `seasonal_naive`.
   - Weather notebook builds the degree-day **data** and saves a parquet;
     the regression model in backtest reads it.

9. **Seasonal amplitude (23%).**
   - = (peak month − trough month) ÷ **yearly average**.
   - Tiny example: Aug 300, Jan 240, yearly avg 260 →
     60 / 260 ≈ 23%.
   - **Not** "August is 23% bigger than January" — that would be
     60 / 240 = 25%.
   - It's the swing as a share of a normal week.

10. **`.clip()` — `lower` vs `upper`.** New 2026-10-23. Used `upper=0` for
    HDD by mistake.
    - `lower=` is the **floor**: anything below gets raised to it.
      `upper=` is the **ceiling**: anything above gets pushed down to it.
      Leave one out → no limit on that side.
    - Tiny example: [5, 12, 8].clip(lower=10) → [10, 12, 10].
    - CDD = (avg − 65).clip(lower=0). HDD = (65 − avg).clip(lower=0).
    - Rule: **flip the subtraction, not the clip.** Both use a floor of 0.
    - `.clip()` doesn't delete or check anything — it pins numbers inside
      a range.

11. **Rehearsal (scratch run before the function).** New 2026-10-28.
    - Run the steps by hand on one fold (fold 1's `train`) in a scratch
      cell. See each step work. Then move the lines into the model
      function.
    - Why: inside a function you can't peek at middle steps, and its
      variables vanish when it ends (see #2).
    - Like cooking a recipe once before cooking it for 12 people.
    - Check question to use: why rehearse on one fold instead of writing
      the function straight away?

12. **The ISO week line, step by step.** New 2026-10-28. Asked to add.
    - `train['week'] = train['ds'].dt.isocalendar()['week']`
    - Each piece hands the next piece something smaller:
      - `train['ds']` → the date column
      - `.dt` → open the date toolbox (only works on a pandas column)
      - `.isocalendar()` → a mini table: `year`, `week`, `day`
      - `['week']` → pick one column by name
      - `train['week'] = ...` → stick it on as a new column
    - Tiny example: 2025-03-16 → (2025, 11, 7). 2025-03-23 → (2025, 12, 7).
      `day` 7 = Sunday.
    - Fresh angle: a funnel. Wide at the top (the whole date), one number
      at the bottom.
    - Check question to use: what does `.isocalendar()` hand back before
      `['week']` — one column or three?

13. **Climatology key: ISO week, not month-day, not 52-row hops.** New
    2026-10-28. Guessed month-day, then 364 days back.
    - Week labels are Sundays. The Sunday drifts about a day each year.
    - Tiny table: 2022-04-03, 2023-04-02, 2024-03-31, 2025-03-30 → four
      different month-days, all ISO week 13.
    - Group by month-day → one row per group → `.mean()` of one row does
      nothing, and no error tells you.
    - Hopping back 52 rows gets **one** past year. That's seasonal-naive.
      Climatology wants **every** past year at once.
    - Rule: group on the thing that stays the same across years.
    - Check question to use: why does grouping by month-day give groups
      of one row?

14. **Last row of `train` = origin, and `iloc[-1]` vs `iloc[:-1]`.** New
    2026-10-28. Said "the 52nd row," then wrote `iloc[:-1]`.
    - `train` always stops at the origin, so its last row is the origin.
      The model works out its 6 forecast dates from there.
    - Tiny example on `[a, b, c]`: `iloc[:-1]` → `[a, b]` (a slice).
      `iloc[-1]` → `c` (one item). `iloc[-2]` → `b`.
    - Rule: colon = a range of rows. No colon = one row.
    - Check question to use: on `[a, b, c, d]`, what does `iloc[:-2]`
      give?

15. **`['name'] = ...` on a Series vs a DataFrame.** New 2026-10-28. Added
    a row by accident.
    - DataFrame → makes a new **column**.
    - Series → makes a new **row** labeled `'name'`.
    - Tiny example: `s = [a, b, c]`, `s['x'] = 5` → `a, b, c, 5`, last one
      labeled `'x'`.
    - `dtype: object` = mixed bag. A sign something got stuffed in.
    - Fix: save the result to its **own** variable.
    - Check question to use: `s` is a Series of 6 dates. After
      `s['week'] = something`, how many rows?

16. **Data flows down into the model, only 6 guesses come up.** New
    2026-10-28. Got it wrong twice (thought folds get joined back; then
    thought climatology lives in `score()`).
    - `score()` cuts `train` → calls `model(train)` → 6 guesses come back.
    - Climatology is built **inside the model**, used, and tossed. Never
      joined back. `all_folds` stacks guesses and answers, not climatology.
    - Fresh angle: 12 students, each tested on a different date, each
      studying only notes from before their date. Each works out "what's
      week 13 usually like?" from **their own** notes. That's why it
      can't be one column in the big table.
    - Check question to use: once `score()` finishes, does fold 1's
      climatology still exist anywhere?

17. **Plumbing + simplest model first.** New 2026-10-28.
    - **Plumbing** = the code that moves data: loading, joins, the
      climatology function, feeding it into `score()`. The pipes, not the
      water. Not "a new piece of data."
    - Rule: when you add new plumbing, run it through the simplest model
      first, so only one thing is new at a time.
    - Degree-day regression **is** a real model (the first one fitted).
      Seasonal-naive is the floor. LightGBM is not "the first real model."
    - Check question to use: why test the weather join on degree-day
      regression instead of LightGBM?

18. **Reading error messages.** New 2026-10-28.
    - Error names tell you the kind of problem; the message names the
      thing.
    - `TypeError: Can only merge Series or DataFrame objects, a <class
      'str'> was passed` → wrong type handed in. Cause: quotes.
      `'sum_of_degree_days'` is a string; `sum_of_degree_days` is the
      table.
    - `AttributeError: 'list' object has no attribute 'isocalendar'` →
      this thing doesn't have that tool. Cause: called it on the whole
      list, not one date.
    - Rule: read what it **wanted** vs what it **got**.
    - Check question to use: `AttributeError: 'Series' object has no
      attribute 'foo'` — what's the problem, in plain words?
