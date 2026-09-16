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

**2026-08-27** - Use Mechanical Permits *Issued*, not Submitted - a filed-but-never-issued
permit is not work, so Issued is the correct demand signal. Supersedes the ambiguous
wording in the original charter.

**2026-08-27** - Default to two era tables (2010–2019, 2020–Present), skipping Before 2010 -
860 weeks is ample; drops one schema seam and removes the 2008–09 construction collapse
as a structural break. Adding the third table requires a stated reason.

**2026-08-27** - Normalize each era table separately before concatenating - separately
maintained tables may differ in column names, date formats, and permit-type vocabulary.

**2026-08-27** - Valuation Target no longer possible - Looking at the both data sets 67is-svtd and 5m3t-xjex I realized that there is no valuation column so for the later change to the model target it will not be happening.

**2026-08-27** — [1:30] — Dataset IDs located and confirmed (67is-svtd 2020–Present, 5m3t-xjex 2010–2019); schemas inspected on both, 31 columns, identical; valuation field confirmed absent, logged, pending item closed.

**2026-09-02** - Using a conda environment with Python 3.12 instead of venv - conda was already installed with Python 3.14 base; creating a 3.12 conda env avoided installing a second Python manager. Deviates from charter §8.

**2026-09-02** — HVAC filter is `permit_type = 'HVAC'` — permit_type is a clean
controlled field with five values (Plumbing, HVAC, Fire Sprinkler, Elevator,
Pressure Vessel), identical vocabulary in both era tables, no nulls. WORK_DESC
free text not needed. Charter §2.1 filter vocabulary item closed.

**2026-09-02** — Pull five columns: permit_nbr, issue_date, zip_code, cd,
permit_sub_type — issue_date is the target's basis per §2.1; zip_code and cd are
the geography preserved for stage 6 per §2.5; permit_sub_type is the
residential/commercial split, also stage 6; permit_nbr is the unique key for
duplicate detection at the era seam.

**2026-09-02** — Socrata returns 1,000 rows silently by default; use explicit
$limit — row counts verified in pandas against server-side count(*): 148,630
(5m3t-xjex) and 91,571 (67is-svtd), both exact. No hidden ceiling found.

**2026-09-03** — Era seam verified clean at the row level — 5m3t-xjex spans
2010-01-03 to 2019-12-31, 67is-svtd spans 2020-01-01 to 2026-08-29. No date
overlap, no rows outside either table's stated era, no rows dated after today.
Zero nulls in issue_date and permit_nbr in both tables; zero duplicate
permit_nbr within either table. Charter §6 seam inspection item closed at the
row level; visual inspection of the plotted series still pending.

**2026-09-03** — Notebook outputs committed rather than stripped — raw data is
gitignored, so a stripped notebook cannot be re-run from the repo and the
outputs are the only record of what was found. Revisit once plots are added:
embedded images bloat the file and produce noisy diffs.

**2026-09-09** — Ended up checking if the seam was clean. Used a line graph from '2019-08':'2020-03' to see if there would be a jump in the data after there was a new data set introduced. In Dec it was nearing 300, by the end of Jan it was 300+, in Feb it was around 390. In the graph there was a huge drop around 2019-2020 ish but it was during March time and not Janaury and had nothing to do with the actual data. The huge crash was due to Covid-19.
**2026-09-09** — Transformed the data series from one permit per row to a weekly series built. Ended up looking at the start and the end of the series to cut of partial weeks. Ended up dropping 2010-01-03 because it was a partial week only including Sunday. Ended up keeping 2026-09-06 because it is true that it did not have any record for Sunday specifically but that is the same for all other weeks as well. Also I ended making all the week end at Sunday. Meaning Starting from Monday until Sunday is what we look at. I got a result of 870 clean weeks.
**2026-09-10** — Gap check passed. Checked the lowest 15 weeks in the data. All of them had an explanation five of them were because of the lock down during Covid-19. The rest are Christmas, Thanksgiving, and New Year weeks which makes sense since there are lower permits are getting issued during those Calendar dates.
**2026-09-10** — Volume check passed. Also the data set has enough Volume to have high Signal To Noise ratio so the data is easier to model and use to predict things. The mean of the data across the 870 weeks is 276.5 permit (pull date on 9/10/2026).
**2026-09-13** — Seasonal amplitude measured at 23% — collapsed the month × year
pivot row-wise to one average per month, then took (max − min) / mean of those 12
numbers. Peak August, trough January. Expressed as a fraction of the yearly average
rather than a raw permit count so the figure is interpretable without knowing the
series level. Note for the write-up: 23% is the gap as a share of the yearly average,
NOT August being 23% above January. Measure is peak-to-trough range, which depends
only on the two extreme months and ignores the other ten — acceptable for a 12-point
seasonal profile, but stated as a known property.

**2026-09-13** — 2026's partial year needs no special handling in the monthly
collapse — data stops in late August, so the Jan–Aug rows include an unusually high
2026 while Sep–Dec rows do not. Dropped the 2026 column and recomputed: amplitude
moved 25% → 23%, peak stayed August, trough stayed January. Two points of movement,
no structural change, so August is a real peak and not an artifact of where the data
ends. General rule applied: missing data distorts an average only when what's missing
is missing for a reason connected to the thing being measured — unequal denominators
alone are harmless.
**2026-09-14** — Seasonality passed on the data set. I set a window from May (5th month) - September (9th month) 12 out of 16 non covid years fit into this pattern. As for 2010-2013 I took a closer look at the table. Those years were just more flat. The peak label was noise, because the columns were flat and idxmax had to pick something. In 2010 the peak beat the runner up by about 16 permits in a column sitting near 260.The data is fine.
**2026-09-14** — The LADBS mechanical permits Issued series is confirmed as v1's dataset, the three gate checks passed, and the fallback is no longer needed so EIA is dead. Volume 2026-09-10, Gap 2026-09-10, and Seasonality 2026-09-14.
**2026-09-16** — Computed the baseline MAE of our actual data. The baseline MAE ended up being 42.8 permits on average miss on the data, this was pulled on 793 weeks. The reason why not all 871 weeks was used is because the first year had nothing to compare to so no calculation could be done. The last 26 weeks are saved for one last run to test out the model. All this was pulled on 2026-09-15. Every model in this project divided by it to score it. A model needs to be under 36.4 permits to hit a MASE of .85 which would pass. This number was calculated based on season-naive .shift(52).


---

## Pending decisions

- [ ] Success target revision, after baseline computed (deadline: data checkpoint)
- [X] Valuation as second target - conditional on field quality assessment
- [ ] Subtype hierarchy - conditional on ≥15 permits/week per series
- [ ] Second project topic - decide December 2026

---

## Weekly checkpoints

**Format:** `YYYY-MM-DD - hours - what was finished`

2026-09-02 - <7> - repo created and pushed, conda env with Python 3.12, week-one packages installed, charter and decision log committed, data/ gitignored
2026-09-02 - <2> - HVAC filter vocabulary confirmed both eras, five-column
pull designed, full row counts verified in pandas, explore.ipynb created
2026-09-03 - <3> - issue_date parsed to datetime on both era tables, date
ranges and null counts verified, permit_nbr uniqueness confirmed per table,
tables concatenated to 240,201 rows
2026-09-03 - <6> - Cross-table permit_nbr uniqueness confirmed on whole_f,
weekly series built with W-SUN resample, partial weeks resolved (870 weeks),
era seam inspected visually and confirmed clean
2026-09-10 - <3:30> - Checked the gap and checked volume.
2026-09-11 - <2> - weekly_labeled built (count, year, month columns),
month × year pivot of average weekly counts produced with pivot_table
2026-09-13 - <1> - monthly_avg collapse built, seasonal amplitude computed
(23%, peak Aug, trough Jan), 2026 partial-year distortion tested and dismissed,
per-year peak months extracted with idxmax
2026-09-14 - <1:30> - Full data set confirmation. Seasonality passed. August was found to be the peak month. The lowest month was January. 2010-2013 data was explained.
2026-09-16 - <2:30> - Found the baseline MAE.


<!-- append one line per week here -->
