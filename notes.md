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
  enough back to leave alone. Revisit once the baseline is scored.

- Seasonality check, second half. Amplitude done (23%, logged). Remaining:
  peak consistency. `peaks = pivot.idxmax(axis=0)` gives the peak month per
  year: 2010:12, 2011:12, 2012:5, 2013:10, 2014:6, 2015:8, 2016:7, 2017:8,
  2018:8, 2019:8, 2020:2, 2021:8, 2022:5, 2023:8, 2024:4, 2025:6, 2026:8.
  Not yet judged. The read: does last year's peak predict this year's —
  that's the bar, because that's what seasonal-naive does. And are the
  off-summer years scattered through the sixteen (noise) or bunched at one
  end (the series behaved differently early on, meaning old data may be
  teaching an outdated pattern). Note 2010–2013 look unlike 2015–2019.
  2020:2 is COVID, already logged. Last of the three §2.1 gate checks.

- Summer 2026 is the highest stretch in the whole series. Not explained yet.

- 2025 sits low across the whole column — 187 in January, most months
  230–270, against 2018 in the 300s. Level question, not seasonality.
  Sits alongside the unexplained summer 2026 high. Related: collapsing the
  pivot down the year axis (`mean(axis=0)`, 2026 excluded) gives a 27%
  spread across years, high 2018, low 2020. Year-to-year level moves more
  than the seasonal swing does — worth keeping in mind before assuming
  seasonality is the dominant structure.

## Next session

- Judge the `peaks` list and call the seasonality gate pass or fail.
- Confirm the dataset in the decision log once seasonality passes.
- Compute the seasonal-naive baseline.
- Decide whether to revise the success target, and log it. One revision
  only, before any model is fit.
- Data checkpoint due Sep 15 — Tuesday.
- Still outstanding: walk through the fetch code in `explore.ipynb` rather
  than just having it work.
