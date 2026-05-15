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
2. **Daily Challenge** — deterministic 5-location set seeded by today's date.
   Plays once per local day; streak tracked.
3. **Practice (random)** — 5 locations picked at random matching the
   1-1-1-2-2 difficulty ramp. Personal-best tracked, doesn't lock.

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

| Section | Approx line | What's there |
|---|---|---|
| LOCATIONS pool | ~840 | 119 entries with trivia |
| HARD_NAMES / MEDIUM_NAMES | ~1100 | tier assignment |
| LEVEL_DATA | ~1170 | 20 curated levels |
| `locId()` | ~1825 | `${lat}_${lon}` identifier |
| `initGlobe()` | ~1270 | D3 projection + state-border fetch |
| `startLevel()` / `startDailyChallenge()` / `startRound()` | ~1770 | game entry points |
| `showSummary()` | ~2000 | end-of-round scoring + save |
| `saveLevelResult()` | ~1600 | first-attempt locks best |
| `exitToHome()` | ~1685 | mid-game ✕ button handler |
