# Schengen 90/180 Calculator

A single-file tracker for the Schengen short-stay rule: **90 days in any rolling
180-day period**. No build step, no dependencies, no server of its own.

Open `index.html` in any browser. Trips are kept in that browser's
`localStorage`, and can optionally sync across all your devices through a
private GitHub repository (see below).

## What it does

- Records past and future trips (country, entry date, exit date).
- Shows days used and days remaining for the 180-day window ending on any date.
- Draws the window as 180 day-cells, so you can see which days still count and
  which have dropped out.
- Charts the rolling total forward against the 90-day ceiling, so you can see
  when the allowance grows back.
- Tests a planned trip against **every day** of the stay and reports the overage,
  the latest legal departure date, and the earliest arrival that fits the full
  requested stay.
- Rolls over on its own. The count is derived from the current calendar day, so
  leaving the page open overnight is fine — it re-renders when the date changes.

## Tentative trips

A trip whose entry date is in the future is labelled **Tentative** and shaded in
the table and the day strip. Nothing needs to be confirmed: as the dates pass it
becomes *In progress*, then *Taken*, because the status is derived from today's
date rather than stored.

The toggle **Count tentative future trips** clips every trip at today, which
answers "where do I stand right now, ignoring everything I have only planned".

## How the allowance grows back

A common misreading is that one day is freed for every day that passes. It isn't.
A day leaves the window exactly **180 days after** it was spent, so the allowance
sits flat and then releases in a block. For 23 May – 29 Jun 2026 plus
28 Aug – 14 Sep 2026:

| Date | Days used | Days free |
|---|---|---|
| 1 Aug 2026 | 38 | 52 |
| 27 Aug 2026 | 38 | 52 |
| 14 Sep 2026 | 56 | 34 |
| 18 Nov 2026 | 56 | 34 |
| 19 Nov 2026 | 55 | 35 |
| 26 Dec 2026 | 18 | 72 |
| 13 Mar 2027 | 0 | 90 |

Nothing is released until 19 Nov, which is 180 days after 23 May.

## Cross-device sync

Data lives in a JSON file in a **private** repository of yours, written through
the GitHub contents API. Nothing passes through any third party.

1. Create a private repository — `schengen-data` is a good name. Leave it empty.
2. **Settings → Developer settings → Personal access tokens → Fine-grained
   tokens → Generate new token.**
3. **Repository access:** *Only select repositories* → pick that one repo.
   **Permissions → Repository permissions → Contents: Read and write.** Nothing
   else is required.
4. Set an expiry you're comfortable with, generate, and copy the token.
5. In the app's **Sync** panel, enter the repo as `owner/name`, keep the path as
   `trips.json`, paste the token, and press **Connect & sync**.
6. Repeat step 5 on each other device. That is the only per-device step.

Syncing happens on load, after every edit, when the tab regains focus, and every
two minutes while visible. **Sync now** forces it.

### How concurrent edits are resolved

Each trip carries its own `updatedAt`. On sync, local and remote lists are merged
per record and the most recent edit of each trip wins, so two devices editing
different trips never lose data. Deleting writes a **tombstone** rather than
removing the record, so a deletion propagates instead of being resurrected by
another device's copy; tombstones are purged after 420 days. Writes carry the
file's blob SHA, so a simultaneous write is detected and retried rather than
silently overwriting.

### Token safety

The token is stored in that browser and sent only to `api.github.com`. Scoped as
above it can read and write that one repository and nothing else, and you can
revoke it on GitHub at any time.

## Hosting it

Sync needs the page served from somewhere that allows outbound requests.

- **Locally:** open `index.html` from disk. Works, including sync.
- **GitHub Pages:** *Settings → Pages → Deploy from a branch* → `main` →
  `/(root)`. The page is then at `https://<user>.github.io/<repo>/schengen/`.
  This repository is public, so host the *page* here and keep the *data* in the
  separate private repo — the page itself holds no secrets.
- **Not the Claude artifact viewer.** That sandbox blocks all outbound requests,
  so a copy opened there stays local to that browser.

## Rules implemented

Regulation (EU) 2016/399 (Schengen Borders Code), Article 6(1)(b):

1. Maximum 90 days of presence in any rolling 180-day period.
2. The window looks backwards: for a given date, count that date plus the
   preceding 179 days. Older days have expired.
3. Day of entry and day of exit each count as a full day.
4. All Schengen states share one budget. Ireland and Cyprus are outside it.
5. A stay is legal only if the limit holds on every day of it, not just the last.
6. Days on a national long-stay (D) visa or a residence permit are excluded —
   don't enter them as trips.

Overlapping trips are merged before counting, so a shared day is never counted
twice; the overlap is flagged in case it was a duplicate entry.

## Not legal advice

Border officers count from the stamps in your passport. Treat this as a planning
aid and keep a margin.
