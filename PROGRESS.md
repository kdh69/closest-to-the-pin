# Progress Log

## v1 snapshot (2026-05-15)

Game was feature-complete as a standalone:

- 119-location pool aligned to `POOL-SPEC.md` (coords exact to spec precision)
- 20 curated levels populated, every `locId` resolves against the pool
- Home screen with Levels grid + Stats tab + Daily Challenge banner
- Practice (random) mode with personal-best
- Fact shown before the guess (in the prompt card, not on reveal)
- US state borders rendered on the globe
- Pinch zoom + auto-zoom after guess
- Mid-game exit button (top-left ✕) — confirms before bailing on level/daily,
  bare exit on practice
- Level best score locks on first completion; replays display "This run: X ·
  Best: Y" without updating the record

## 2026-06-03 — Embedded in Daily Dynasty + daily redesign

**Done**

- This game is now embedded in the **Daily Dynasty** app (`../pmo-daily-dynasty`) as
  one game in a multi-game daily arcade. It's served from that app's
  `public/games/pin/index.html` inside an `<iframe>`.
- Added `reportDailyToHost(score, MAX_TOTAL, streak)` (called at the end of
  `saveDailyResult()`): when embedded, it `postMessage`s the daily result to the host,
  which posts it to a shared league leaderboard. Harmless no-op when opened standalone,
  so GitHub Pages is unaffected. **Keep this hook intact.**
- `index.html` here is the **source of truth**. The Dynasty app re-pulls it with
  `npm run sync:pin` (run from the Dynasty repo) — a pure file copy, since both copies
  carry the identical hook.
- **Home redesign shipped.** Home now opens to a centered Daily hero card (big icon,
  date, play button). Stats tab fully removed (CSS, DOM, `renderStats()` call site
  cleared). Levels demoted to a small "▦ Levels" button that swaps in a secondary view
  with a "← Home" header — `renderLevelGrid()` still drives the grid.
- **Daily: 23 guaranteed-unique days, then random — shipped.** `getDailyLocations()` is
  re-anchored to launch epoch `DAILY_EPOCH = '2026-06-04'` (Day 1). It buckets the pool
  by difficulty, **dedupes each tier by `locId`** (guards the same-coord collision) and
  sorts by `locId` for a stable base order. For the first
  `CURATED_DAYS = min(floor(easy/2), medium, floor(hard/2)) = 23` days it deterministically
  shuffles each tier with a FIXED seed (`xor32(fnv32('ctp-daily-shuffle-v1'))`) and hands
  out non-overlapping slices (`easy[2d],easy[2d+1]`, `medium[d]`, `hard[2d],hard[2d+1]`)
  in ramp order, so **no location repeats across 23 days**. From day 24 on it falls back
  to per-day seeded random (`'ctp-daily-rand-' + dayIndex`, repeats allowed).
  - **Why 23:** each daily is 2 easy + 1 medium + 2 hard. After dedupe the live pool is
    49 easy / 23 medium / 46 hard; medium (23/1) and hard (46/2) both cap at 23. That's
    the max fully-unique streak (computed from live tier sizes, not hardcoded).
  - **Verified via Node sim:** days 0–22 give 115 distinct `locId`s (zero repeats), every
    set is a valid ramp-order 2-easy/1-medium/2-hard; day 23+ is random with no errors;
    the same day is stable across reloads.
  - **Rationale / scope:** this finite curated-then-random model suits a *map/location*
    daily where "don't repeat a place" matters. It is NOT the model for a procedural
    *random-gen* daily (e.g. the NBA draft game), which makes a fresh board every day and
    is effectively unlimited.

## Recent commits

| Commit | What |
|---|---|
| (uncommitted) | Home redesign (Daily hero, Stats removed, Levels demoted) + 23-day no-repeat daily (epoch `2026-06-04`) + `reportDailyToHost` hook + PROGRESS/PROJECT rewrite |
| `b182503` | Track `POOL-SPEC.md` and `.docx` as source-of-truth |
| `3e96706` | Add `PROJECT.md` and `PROGRESS.md` |
| `87ebcb5` | Mid-game exit + first-completion-locks-best |
| `903b525` | `LEVEL_DATA` populated, 119 pool coords aligned to spec |
| `9fa837e` | Promoted current build to `index.html` (was `closest-to-the-pin.html`) |
| `002aa91` | Pool tier alignment, 10 drifted coords fixed, ← Home on practice summary |
| `cc1afb6` | Home screen visibility fix + favicon 404 fix |
| `30fd1d0` | Home screen with daily / level grid / stats |
| `8053500` | Scoring, seen tracking, pool overhaul, trivia rewrite |
| `d8271ec` | Pinch zoom, auto-zoom-after-guess, US state lines |

## Known quirks

- **Jordan Spieth + Lee Trevino share coords** (`32.7767, -96.7970` — both
  Dallas in `POOL-SPEC.md`). `locId()` collides, so Level 7's intended
  Lee Trevino entry resolves to Jordan Spieth at runtime. Mechanical play is
  unaffected; only the displayed name/trivia differ from spec intent. The daily
  generator now dedupes each tier by `locId`, so this pair counts as one slot there
  (live medium tier is 23 after dedupe, which sets `CURATED_DAYS`).
- **Comments in `LOCATIONS` use minor naming variants** of spec
  (e.g. spec "Royal Portrush (Dunluce)" → HTML "Royal Portrush (Dunluce
  Links)"). Same locations, same coords — just label drift.

## What's next (open ideas — not committed)

- Verify the 20 levels mechanically satisfy the spec's geographic rules:
  no two locations within 500 mi inside a level; no two levels share 4+
  locations. Never been programmatically checked.
- Consider styled exit-confirm modal instead of native `confirm()`.
- Local daily-history log (per-day score archive) — beyond today's score
  and streak. The Dynasty host already retains cross-day data via its
  leaderboard, so this is only useful for standalone play.

## Spec docs

- `POOL-SPEC.md` — source-of-truth, markdown, full 119 list with coords.
- `Closest-to-the-Pin-Pool-Spec.docx` — same content, Word format. Kept in
  sync manually.
