# Progress Log

## Current state (2026-05-15)

Game is feature-complete for v1:

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

## Recent commits

| Commit | What |
|---|---|
| `87ebcb5` | Mid-game exit + first-completion-locks-best |
| `903b525` | LEVEL_DATA populated, 119 pool coords aligned to spec |
| `9fa837e` | Promoted current build to `index.html` (was `closest-to-the-pin.html`) |
| `002aa91` | Pool tier alignment, 10 drifted coords fixed, ← Home button on practice summary |
| `cc1afb6` | Home screen visibility fix + favicon 404 fix |
| `30fd1d0` | Home screen with daily / level grid / stats |
| `8053500` | Scoring, seen tracking, pool overhaul, trivia rewrite |
| `d8271ec` | Pinch zoom, auto-zoom-after-guess, US state lines |

## Known quirks

- **Jordan Spieth + Lee Trevino share coords** (`32.7767, -96.7970` — both
  Dallas in `POOL-SPEC.md`). `locId()` collides, so Level 7's intended
  Lee Trevino entry resolves to Jordan Spieth at runtime. Mechanical play is
  unaffected; only the displayed name/trivia differ from spec intent.
- **Comments in `LOCATIONS` use minor naming variants** of spec
  (e.g. spec "Royal Portrush (Dunluce)" → HTML "Royal Portrush (Dunluce
  Links)"). Same locations, same coords — just label drift.

## What's next (open ideas — not committed)

- Verify the 20 levels meet the spec's "no two within 500 mi per level" rule.
  Spec says to check; never been mechanically verified.
- Verify no two levels share 4+ locations.
- Consider styled exit-confirm modal instead of native `confirm()`.
- Track daily-challenge history (currently only today's score / streak — no
  log of past days).
- Stats tab could show per-category accuracy or worst-performing level.

## Spec docs

- `POOL-SPEC.md` — source-of-truth, markdown, full 119 list with coords.
- `Closest-to-the-Pin-Pool-Spec.docx` — same content, Word format. Kept in
  sync manually.
