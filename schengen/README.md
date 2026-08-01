# Schengen 90/180 Calculator

A single-file, offline tracker for the Schengen short-stay rule: **90 days in any
rolling 180-day period**.

Open `index.html` in any browser — there is no build step, no server and no
network access. Trips are stored in that browser's `localStorage`, so they
persist between visits on the same device. Use **Export** / **Import** to move
data or keep a backup.

## What it does

- Records past and future trips (country, entry date, exit date).
- Shows how many of the 90 days are used and how many remain for the 180-day
  window ending on any date you choose.
- Draws the window as 180 day-cells so you can see exactly which days are still
  counting and which have dropped out.
- Charts the rolling total forward, against the 90-day ceiling, so you can see
  when days come back.
- Tests a planned trip against **every day** of the stay and reports the overage
  in days, the latest legal departure date, and the earliest arrival date that
  fits the full requested stay.

## Rules implemented

Regulation (EU) 2016/399 (Schengen Borders Code), Article 6(1)(b):

1. Maximum 90 days of presence in any rolling 180-day period.
2. The window is backward-looking: for a given date, count that date plus the
   preceding 179 days. Older days have already expired.
3. The day of entry and the day of exit each count as a full day.
4. All Schengen states share one budget. Ireland and Cyprus are outside it.
5. A stay is legal only if the limit holds on every day of it, not just the last.
6. Days spent on a national long-stay (D) visa or a residence permit are excluded
   — simply don't enter them as trips.

Overlapping trips are merged before counting, so a shared day is never counted
twice; the app flags the overlap so you can check for a duplicate entry.

## Not legal advice

Border officers count from the stamps in your passport. Treat this as a planning
aid and keep a margin.
