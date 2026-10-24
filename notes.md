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

- Weather archive (ERA5-Land) runs about a week behind today. On the
  2026-10-07 pull the last 7 days came back NaN (unpublished, not missing).
  On the 2026-10-23 pull they came back filled, 0 NaN anywhere, because the
  URL's end date (Oct 7) was already past the ~7-day lag. Rule: compare the
  requested end date against "today minus the lag." Doesn't matter now.
  Will matter in January when deployed and pulling fresh. Decide then.

- Weather data depends on the pull date, same as permits. Current
  `sums_of_degree_days.parquet` was built from the 2026-10-23 pull. It's a
  snapshot: if the URL or end date changes, re-run `weather.ipynb` and
  re-save.

- Hot-day CDD spot check was skipped on 2026-10-23. Only a cold day (row 0)
  was checked by hand, and the "CDD > 0 and HDD > 0" check can't catch a
  flipped sign. Untested: pull the highest `daily_avg` row and confirm
  CDD = avg − 65, HDD = 0.

## Next session

- **Timeline:** Weather parquet done 2026-10-23. Degree-day regression (Oct
  20 milestone) still not done — that's next. **Nov 3 MVP is the
  load-bearing date** (LightGBM + baseline + evaluation), 11 days out as of
  2026-10-23. Degree-day regression is not part of the MVP definition —
  decide whether it stays ahead of LightGBM.
- **Log in decision-log.md if not done 2026-10-23:**
  - Weekly degree days: daily CDD/HDD per spot → equal-weight average per
    day → weekly **sum** (W-SUN) → partial weeks dropped (first week
    2010-01-03 had 1 day, last week 2026-10-11 had 3 days) → 874 weeks,
    saved to `data/sums_of_degree_days.parquet`, pull date 2026-10-23.
    Write the reason for *sum* in my own words.
  - Date column named `ds` to match the permit parquet so the join is one
    clean line.
- Weather decisions logged 2026-10-07 (6 entries: climatology for all
  horizons, pre-origin climatology, 3 spots + equal weights, ERA5-Land,
  degree-day settings, weather.ipynb). Fix flagged typos if not done.
- State of `backtest.ipynb` (clean Restart + Run All passed 2026-10-07):
  `score(model)` loops `range(len(origins))`, cuts both `train` and `akey`
  from `used_set`, stores guesses as `predicted` → column `prediction`,
  MAE via `groupby('horizon')['miss'].mean()`. `baseline_MAE =
  score(seasonal_naive)`, `target_horizon = baseline_MAE * .85`
  (h4 bar 29.8). Scratch cells deleted.
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
- **Cleanup in `weather.ipynb`:** cells that overwrite their own input
  (`three_average_week = three_average_week.reset_index()`, and the
  rename cell) are rerun-unsafe — fold into one line writing a new name
  (e.g. `weekly_dd`). Names `three_average` / `taw` / `three_average_week`
  are vague. Delete scratch cells (failed concat attempts, `.size()` check,
  early test loops, `first_weather_df`). Then Restart + Run All and confirm
  (874, 3) again.
- **Pick up at:** degree-day regression in `backtest.ipynb`. Load
  `sums_of_degree_days.parquet`, join to permits on `ds`. The model is a
  function next to `seasonal_naive`: takes `train`, returns 6 guesses as a
  plain list. Open question to start: how to build climatology for the 6
  forecast weeks using **only years before the fold's origin** (no
  leakage). Calendar features per charter §3 come after that.
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

## Review style

How reviews run. Claude follows this every review session. Update it when
something isn't clicking.

- **One question at a time.** Show progress: *Topic X/N · Block Y*.
- **I answer first.** Claude doesn't hint before I try.
- **If I'm right:** one line confirming, then move on. No follow-up, no
  extra depth. Don't spend energy on things I already know.
- **If I'm wrong or half-right, explain in this order:**
  1. Say it's wrong in one line.
  2. **Tiny picture example:** a made-up table of 3–5 rows I can see,
     not the real 72-row table.
  3. **What happens,** shown on that tiny table.
  4. **One-line rule** to remember.
  5. One simple check question. If I get it, move on.
- **Plain words.** Short sentences. No new terms inside an explanation.
  Everyday comparison first if it helps (outlets, packages, pixels,
  fabric seams).
- **Give context in the question** when it needs code or numbers I might
  not have memorized: show the line, the table, or the setup. I won't
  have the code in front of me.
- **Wrong or shaky topics** go on a comeback list, re-asked later with a
  fresh example. If I say "don't add it," don't.
- **Don't ask** date-counting by hand (I check those with code).
- If I've explicitly asked for a better explanation on a topic, give it
  a new angle, not the same words again.

## Next review

Comeback list from the 2026-10-22 review (42 topics done, these 9 were
wrong or shaky), plus items I asked to add during the 2026-10-23 build
session. Use the Review style above. Each has a fresh angle to try.

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

2. **Kernel memory + function scope.**
   - A loop overwrites a variable every pass; after it ends, the variable
     holds the **last** pass (fold 12's `train`).
   - Variables made **inside a function** vanish when it ends. `train` now
     lives inside `score()`, so typing `train` in a new cell → NameError.
   - Fix for stale memory: Restart kernel → Run All; counts start at [1].

3. **Regression test — the definition, not just the habit.**
   - Definition: change code, then check it still gives the **old, known
     answer**. "Regression" = going backwards (code breaking what used to
     work). Not the regression model.
   - Habit: call the numbers before running.
   - Example: rewrote `score()` with `.mean()`, checked it still gave
     22.2 / 28.7 / 36.8 / 35.1 / 25.4 / 25.8.

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
