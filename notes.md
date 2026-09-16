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
  2026-09-15, since `67is-svtd` is live. Any recorded count needs a pull
  date attached.

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

## Next session

- Decide whether to revise the success target, and log it. One revision
  only, before any model is fit. This is the last open item on the Sep 15
  data checkpoint.
- Harness milestone due Sep 30: rolling-origin backtest that scores any
  model, seasonal-naive scored through it.
- Still outstanding: walk through the fetch code in `explore.ipynb` rather
  than just having it work.
