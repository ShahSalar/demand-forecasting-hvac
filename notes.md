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

- `issue_date` arrives from the Socrata API as a string, not a datetime.
  Needs `pd.to_datetime` before any weekly bucketing can happen.

- Future issue dates in `67is-svtd` (2020–Present). Saw `2026-04-28` in the
  first five rows; today is 2026-09-02. Cause unknown. Check how many rows are
  dated after today and what they look like before building the weekly series.

## Next session

- Walk through `explore.ipynb` line by line — understand the fetch code rather
  than just having it work.
