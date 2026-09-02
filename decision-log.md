# Decision Log

Every decision gets a date and a one-line reason. Append only - never edit or delete a
past entry. If you change your mind, add a new entry that supersedes the old one.

This file becomes the backbone of the write-up. Keeping it costs a minute a week and
saves a week in December.

**Format:** `YYYY-MM-DD - decision - reason`

---

## Scope decisions

**2026-08-26** - Project is demand forecasting, single project, depth over breadth -
a system that does one thing at a level nobody expects beats one that does six things
adequately.

**2026-08-26** - v1 built on public LADBS permit data rather than company data - company
records are not reliable or complete enough; company version deferred to v2 after
enough clean history is collected.

**2026-08-26** - Target is weekly permit count, not valuation - count is the cleanest
field in the dataset and gets to a working forecast fastest; valuation quality unknown
until inspected.

**2026-08-26** - Weekly granularity, Monday-start weeks labeled by Sunday end - LADBS
issues permits Mon–Fri so weekends are structural zeros; Monday-start matches ISO weeks
for clean weather joins and matches the company's Mon–Sat work week for v2 continuity.

**2026-08-26** - Horizon set to 6 weeks, headline claim at week 4 - six weeks matches
equipment ordering and hiring lead times in the trade; week 4 is defensible as the core
planning window without cherry-picking the easiest horizon.

**2026-08-26** - v1 forecasts citywide total only; geography and subtype columns
preserved but unmodeled - splitting costs signal, and the threshold for splitting
(≥15/week per series) cannot be evaluated until the data is in hand.

**2026-08-26** - Two-dimensional hierarchy (geography × subtype) permanently out of
scope - resulting series would be too sparse, and this is the most likely place for the
project to sprawl.

**2026-08-26** - Success target set provisionally at MASE ≤ 0.85 at week 4, coverage
85–95% - 15% improvement over seasonal-naive is realistic for a series with strong
yearly seasonality already captured by the baseline. One revision permitted after
computing the baseline, before any model fitting.

**2026-08-26** - Backtest: rolling origin, expanding window, 12 folds stepped 4 weeks,
no gap, 26-week untouched holdout - locked before any results exist, so the eventual
number is trustworthy. Revisable only for methodological reasons, never on scores.

**2026-08-26** - v1 deadline December 15, 2026; MVP by November 3 - self-imposed, with
real slack behind it (applications are Fall 2027). The January 2027 deployment date
depends on it.

**2026-08-26** - Repo public from first commit - v1 uses only public data; visible
commit history is evidence of sustained work and enforces tidiness.

2026-08-27 - Use Mechanical Permits *Issued*, not Submitted - a filed-but-never-issued
permit is not work, so Issued is the correct demand signal. Supersedes the ambiguous
wording in the original charter.

2026-08-27 - Default to two era tables (2010–2019, 2020–Present), skipping Before 2010 -
860 weeks is ample; drops one schema seam and removes the 2008–09 construction collapse
as a structural break. Adding the third table requires a stated reason.

2026-08-27 - Normalize each era table separately before concatenating - separately
maintained tables may differ in column names, date formats, and permit-type vocabulary.

2026-08-27 - Valuation Target no longer possible - Looking at the both data sets 67is-svtd and 5m3t-xjex I realized that there is no valuation column so for the later change to the model target it will not be happening.

2026-08-27 — [1:30] — Dataset IDs located and confirmed (67is-svtd 2020–Present, 5m3t-xjex 2010–2019); schemas inspected on both, 31 columns, identical; valuation field confirmed absent, logged, pending item closed.

---

## Pending decisions

- [ ] Success target revision, after baseline computed (deadline: data checkpoint)
- [X] Valuation as second target - conditional on field quality assessment
- [ ] Subtype hierarchy - conditional on ≥15 permits/week per series
- [ ] Second project topic - decide December 2026

---

## Weekly checkpoints

**Format:** `YYYY-MM-DD - hours - what was finished`

<!-- append one line per week here -->
