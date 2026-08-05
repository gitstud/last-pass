# Last Pass

Endless slingshot climber. Landscape or portrait mobile web game, plain HTML5
canvas, zero dependencies, one file (`index.html`). Prototyped Aug 2-4 2026
across five pivots; the exploration history lives in `~/dev/last-pass-game`
(git repo, `prototypes/` folder).

## The game

A ninja (gold ball placeholder for now) climbs an endless tower of switchback
platforms. Drag anywhere on screen, release to sling — the pull vector is the
shot, a dotted preview simulates the true trajectory live (including while
airborne, so you queue the next sling mid-flight and release at the right
moment: that timing is the core skill).

- **Three slings per landing, diminishing**: each air sling launches at
  `airDecay` (0.8) × the previous power. Touching anything reloads all three.
- **Map scale is defined in jump units**: `TIERS = [330, 560, 720]` px rises =
  clean single / committed double / crisp triple, set at ~75% of the measured
  chain ceilings (444 / 733 / 918 px). If `gravity`, `maxThrow`, or `airDecay`
  change, re-measure ceilings with the real integrator and reset TIERS.
- **No deaths.** Worst case you fall all the way to the ground floor. Loss of
  altitude IS the punishment currency.
- **Zones**: sky palette lerps and zone names announce as you climb
  (Courtyard → Ramparts → Cloudbank → High Sky → Starfield → Moon's Reach).

## Physics feel (user-dialed on device, do not casually change)

gravity 2300 · slingPower 7 · maxThrow 1500 · restitution 0.4 · damping 0.2 ·
groundFric 6 (character skid: 1000px/s stops in ~0.4s) · ballR 31 · jumps 3 ·
airDecay 0.8. Tuning philosophy: **control and readability over speed; weight
over zip**. Difficulty comes from generation (tier mix, gaps), never from
re-adding energy to the ball. The ⚙ panel exposes all knobs live on device.

## Vision / roadmap (agreed with Max)

1. Sprite ninja character — 3 poses: aim-crouch, flight-tuck, landing-skid.
2. Stealth, not combat: sentries/searchlights with vision cones. Being spotted
   never kills — enemies knock you off platforms. Stealth failure converts to
   fall risk; stillness (the natural resting state) is safety.
3. Catch floors every ~250m: arrest mega-falls, house sentries/NPCs/anchors.
4. Mission gates every 10,000 ft over one endless tower; narrative in authored
   "landmark floors" between procedural stretches.
5. Monetization: content + cosmetics IAP (zone packs, ninja skins/trails).
   Max explicitly rejected selling checkpoints/progress-recovery ("seems like
   I make you lose on purpose"). Coins (persistent, `lp-coins`) are the soft
   currency. Working framework (Aug 5): (1) cosmetics as identity — skins,
   trails, sling/preview styles, celebration themes, priced in coins, coin
   bundles as IAP; (2) content packs — themed zone/chapter expansions,
   one-time purchases; (3) a cosmetic-only "climb pass" keyed to peak
   altitude per season; (4) rewarded-only ads as an optional lever, never
   interstitials; (5) NEVER: energy timers, pay-for-power, loss-recovery
   purchases. Sequence: instrument retention first (PM collaborator), ship
   monetization only after D1/D7 proves the loop.
   RULE: no pausing inside the redo window — opening settings forfeits the
   offer (the moment exists only in real time; slow-mo IS the thinking time).
   Backlog: an opt-in "calm redo window" accessibility toggle (extended or
   frozen timer) once a player-facing settings screen exists — default stays
   brutal.
   VALIDATED DESIGN FINDING (Aug 5, Max): the redo prompt's 3s decision window
   is an anticipated-regret engine — hesitating until it closes carries the
   fall AND the regret; keep it brutal for SPENDING redos. HARD RULE: never
   put a countdown on a MONEY decision (that's a pressure sale) — if buying
   ever goes real, the purchase path must escape the timer (e.g., offer
   persists briefly after landing).
   REDO economy (candidate, Aug 5): the redo mechanic (undo a >120m fall) ships
   FREE — 3 per climb — while playtesting. If monetized: sell PACKS (~5/$1) in
   a calm store moment, never a buy button inside the falling moment (that's
   the "make you lose on purpose" line). Free 3 refresh every climb; purchased
   redos are a persistent wallet consumed only after the free ones — packs
   thus target deep single runs, not everyday play. Max's read: most players
   just want to climb to pass time — redo packs are minority revenue;
   cosmetics/content remain the spine.
6. "Reverse pachinko" is the pitch line. Naming direction: Chinese —
   弹弓 dàngōng ("elastic bow"/slingshot); explore variations.

## Release ritual (standing rule from Max)

**Every deploy**: add a CHANGELOG entry in `index.html` (newest first), bump
`VERSION`, and curate `highlights` — the short lines playtesters see on the
title screen. The full changelog lives behind "What's new" in the ⚙ panel;
a red dot on the gear marks unseen versions (`lp-seen-ver`).

## Deployment

**Permanent playtest URL: https://gitstud.github.io/last-pass/** — GitHub
Pages off `main` (repo `gitstud/last-pass`). Deploy = `git push` (live in
~1-2 min). ngrok/LAN serving remain useful for instant-refresh iteration;
Pages is the link you share.

## Dev workflow

- Serve locally: no-cache python server on :8000 (see scratchpad `serve.py`
  pattern — `Cache-Control: no-store`), phone on same wifi hits the Mac's LAN
  IP. ngrok (installed + authed) for remote playtesters: `ngrok http 8000`.
- Headless testing: extract the script with awk, stub the DOM with a recursive
  Proxy, `eval` + drive functions directly in node. Used for: measuring jump
  apexes with the real integrator, generation fuzzing, flood-fill
  climbability proofs (see old repo's scratch harnesses).
- If generation ever gains obstacles that could seal a route, use the
  witness/flood-fill verification pattern from the prototypes: each chunk
  returns a known-open point, prove reachability from the previous one with a
  ball-radius-inflated flood fill, reroll on failure. Never ship "probably
  passable".
- localStorage keys: `lp-zig-best` (best altitude), `lp-coins`.
