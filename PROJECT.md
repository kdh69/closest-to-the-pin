# Closest to the Pin

A globe-based golf location guessing game. The player is shown a name (a famous
course, a player's birthplace, an equipment HQ, etc.) plus a one-paragraph
"fact" about it, and they tap on a 3D globe to mark where they think it is.
Score is awarded by proximity with exponential decay.

## Stack & deployment

- Single static file: `index.html` (everything inline — HTML, CSS, JS, world
  TopoJSON data, the 119-location pool, and all level data).
- Runtime: vanilla JS + D3 (`d3.geoOrthographic` projection, drag/pinch zoom).
- US state borders are fetched at runtime from `us-atlas@3` via jsDelivr.
- No build step. Open `index.html` in a browser, or push to any static host.
  Hosted at https://github.com/kdh69/closest-to-the-pin (GitHub Pages serves
  `index.html` at the root URL).

## Game modes

1. **Levels** — 20 curated levels, each a sequence of 5 locations (1 easy, 1
   easy, 1 medium, 2 hard). 1000 max pts per level. Stars: 1 (<60%), 2 (60–85%),
   3 (≥85%). **Best score locks on first completion** so memory can't game it;
   replays are allowed for fun but don't update the record.
2. **Daily Challenge** — the home hero card; deterministic 5-location set, same for
   everyone, plays once per local day; streak tracked.
   - Anchored to `DAILY_EPOCH = '2026-06-04'` (Day 1). The first
     `CURATED_DAYS = min(floor(easy/2), medium, floor(hard/2)) = 23` days are a fixed,
     no-repeat sequence (each location used at most once), built by fixed-seed
     (`'ctp-daily-shuffle-v1'`) per-tier shuffles sliced as
     `easy[2d],easy[2d+1] / medium[d] / hard[2d],hard[2d+1]` in ramp order.
   - From day 24 on it falls back to per-day seeded random (`'ctp-daily-rand-'+dayIndex`,
     repeats allowed). 23 = the max fully-unique streak for a 2-easy/1-medium/2-hard daily
     against the deduped 49/23/46 pool (capped by medium 23/1 and hard 46/2). Finite
     curated model is specific to a *map* daily, not a procedural random-gen one.
3. **Practice (random)** — 5 locations picked at random matching the
   1-1-1-2-2 difficulty ramp. Personal-best tracked, doesn't lock.

## Daily Dynasty integration

This game is embedded as one tab in the **Daily Dynasty** multi-game app
(`../pmo-daily-dynasty`), served from that app's `public/games/pin/index.html` in an
`<iframe>`.

- **Score reporting:** `saveDailyResult()` calls `reportDailyToHost(score, MAX_TOTAL,
  streak)`, which `postMessage`s `{ type: 'pin:dailyResult', score, max, streak, day }`
  to `window.parent`. The host (`components/PinGame.jsx`) catches it and POSTs to its
  `/api/score` shared leaderboard. No-op when opened standalone — keep it intact.
- **Source of truth:** this repo's `index.html`. Sync into Dynasty with `npm run sync:pin`
  (run from the Dynasty repo); it's a pure copy because both files carry the same hook.
- The embed currently sits behind a commented-out entry in Dynasty's `lib/games.js`
  (uncomment to re-enable the ⛳ tab).

## Data model

- **`LOCATIONS`** — 119 entries, each `{ name, category, country, lat, lon, trivia }`.
  - 50 easy / 23 medium / 46 hard (tier assigned via `HARD_NAMES` and
    `MEDIUM_NAMES` sets; everything else defaults to easy).
  - 5 categories: Famous Course, Championship Venue, Player Birthplace,
    Golf Landmark, Equipment HQ.
- **`LEVEL_DATA`** — 20 entries, each `{ id, locs: [locId, ...] }` where `locId`
  is the string `"{lat}_{lon}"` matching a pool entry.
- **`POOL-SPEC.md`** — source of truth for the 119-entry pool. All coords in
  `LOCATIONS` are kept at the spec's precision so `locId()` lookups resolve.

## Scoring

- Distance → points via `exp(-miles / 1500)` × base × multiplier.
- Position multipliers per round: `[1, 1, 2, 3, 3]` → max 1000 pts/level.

## File layout

```
index.html                       # the entire app
POOL-SPEC.md                     # source-of-truth for the 119 locations
Closest-to-the-Pin-Pool-Spec.docx  # same spec, Word-formatted
PROJECT.md                       # this file
PROGRESS.md                      # status & log
```

## Key code landmarks (inside index.html)

| Section | Line | What's there |
|---|---|---|
| `LOCATIONS` pool | ~865 | 119 entries with trivia |
| `HARD_NAMES` / `MEDIUM_NAMES` | ~1145 | tier assignment |
| `LEVEL_DATA` | ~1193 | 20 curated levels |
| `locId()` | ~1885 | `${lat}_${lon}` identifier |
| `initGlobe()` | ~1285 | D3 projection + state-border fetch |
| `startLevel()` / `startDailyChallenge()` / `startRound()` | ~1853 | game entry points |
| `getDailyLocations()` | ~1699 | seeded daily picker: epoch + 23-day no-repeat curated, then per-day random |
| `showSummary()` | ~2062 | end-of-round scoring + save |
| `saveLevelResult()` | ~1644 | first-attempt locks best |
| `saveDailyResult()` | ~1657 | per-day write, streak bump, calls `reportDailyToHost()` |
| `reportDailyToHost()` | ~1676 | embed hook — `postMessage` to parent, no-op standalone |
| `exitToHome()` | ~1742 | mid-game ✕ button handler |
