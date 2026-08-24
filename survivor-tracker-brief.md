# Project Brief: NFL Survivor Pool Tracker

## What this is
A personal-use, single-page web app for playing NFL Survivor pools. You pick one team
per week to win straight-up; picking the same team twice is illegal, so the app locks
a team out of every future week once you've used them. It also estimates win
probability per matchup and recommends which week to "spend" each remaining team.

A working v1 already exists as a single self-contained HTML file (vanilla JS, no
build step, no frameworks). This brief describes what's built and what to extend.

## Starting point
The existing file is attached alongside this brief (`survivor-tracker.html`). Open
it directly — it's a complete, working artifact. Use it as the base to build from
rather than starting over.

## Current feature set (v1)
- Full 2026 NFL regular season schedule hardcoded (18 weeks, all 272 games, all bye
  weeks computed from the schedule data — not hardcoded separately).
- Tap a team to pick them for the current week. That team becomes disabled (greyed
  out, "used" tag) in every other week automatically.
- A horizontal "yard line" progress tracker across the top (18 marks, one per week)
  that fills in as picks are made; tap any mark to jump to that week.
- Status bar: picks made, weeks remaining, teams still alive.
- Win probability model: preseason Vegas win totals converted to a power rating,
  combined with a fixed home-field-advantage constant, run through a logistic
  function to produce a win % for each team in each matchup. Shown next to every
  team button; favorites (≥65%) are visually highlighted; the single best
  available pick for the currently-viewed week gets a "★ top pick" badge.
- "Best future week per team" planner: for every team you still have available,
  finds the highest-win% week remaining on their schedule and lists it, sorted
  best to worst, with a tap-to-jump row.
- "Teams still in play" grid: all 32 teams, struck through once used.
- Picks persist automatically via `window.storage` (Claude artifact persistent
  storage API — personal/non-shared scope), so the plan survives across sessions.
- Reset-all-picks control (with confirmation).

## Design system already in place
Keep visual continuity with this unless there's a good reason to deviate:
- **Palette**: deep gridiron green background (#101d16), card surface (#1c3126),
  chalk-white text (#f1ede1), sage muted text (#8fac93), amber accent for
  favorites/highlights (#dba63f), red for used/eliminated (#c4523a).
- **Type**: Anton (condensed display) for headers/week numbers, Inter for body
  text, JetBrains Mono for stats, abbreviations, and data labels.
- **Signature element**: the yard-line progress strip at the top — this is the
  one distinctive visual idea and should stay central to the design as the app
  grows, not get crowded out by new features.

## Known limitations to fix or improve
1. **Win probability model is crude.** It's built from a single static snapshot
   of preseason win totals (checked mid-August 2026) and a fixed home-field
   constant — not live weekly spreads. It will get less accurate as the season
   progresses and real form/injuries diverge from preseason expectations.
   Priority fix: pull real weekly point spreads once available (see Data sources
   below) and fall back to the win-total model only for future weeks that don't
   have posted lines yet.
2. **No pool-size or rule customization.** Currently assumes a standard
   single-entry, single-team-per-week, straight survivor format. Real pools vary:
   double-pick weeks, strikes/buy-backs, "beat the spread" variants, multiple
   entries per person. Not required for v1 but worth a settings panel later.
3. **No public pick popularity data.** Serious survivor strategy weighs not just
   win probability but *how many other entrants picked that team* — going
   contrarian in large pools has value even at lower win%. This app currently
   ignores pool dynamics entirely.
4. **Single device/browser only.** `window.storage` in personal (non-shared) mode
   is scoped to this artifact session context — confirm before relying on it
   long-term; consider whether a login-free but durable storage story is needed
   if this becomes a real multi-week tool used across devices.
5. **No mid-season data refresh path.** The schedule and win totals are baked in
   as static JS objects. There's no update mechanism if this needs to reflect
   actual results/injuries as the season unfolds.

## Data sources used to build v1 (for reference / refresh)
- **Schedule**: pro-football-reference.com/years/2026/games.htm (all 18 weeks,
  home/away, dates) — transcribed directly into the `SCHEDULE` JS array.
- **Win totals**: legalsportsreport.com/nfl/odds/win-totals (checked ~Aug 12,
  2026) — one consensus number per team, stored in `WIN_TOTALS`.
- Both are single-snapshot manual pulls, not live API feeds. There is no
  automated refresh currently.

## Suggested build priorities for Claude Code, roughly in order
1. **Live odds integration.** Replace/augment the static win-total model with
   real weekly point spreads as they're posted (a sports odds API — e.g. The
   Odds API, or scraping a public spread aggregator — would work; pick whichever
   fits your existing tooling/budget). Recompute win% per matchup weekly instead
   of once from preseason numbers.
2. **Actual results ingestion.** Once games are played, mark them final and lock
   past weeks so history is accurate (currently there's no concept of "this game
   already happened").
3. **Pool rules panel.** Let the user configure: pool size (affects how
   aggressively to go against the popular pick), double-pick weeks, strikes
   allowed, whether ties count as losses.
4. **Public pick popularity signal**, if a data source is available — even a
   rough heuristic (e.g. treat the biggest home favorite each week as "the
   chalk everyone's taking") would materially improve the strategic planner.
5. **Multi-entry support**, so someone running 2–3 entries in the same pool can
   plan them together without one entry's picks blocking the others.
6. Polish pass on the "best future week" planner — right now it's a simple
   greedy per-team lookahead; a smarter version would solve the whole-season
   allocation problem (which team should go in which week to maximize total
   survival probability across all 18 weeks), which is a real optimization
   problem, not just a per-team lookup.

## Explicit non-goals for now
- No account system / login — this is a personal single-user tool.
- No real-money or pool-hosting features (inviting others, tracking a group's
  picks) — that's a different, much bigger product (see PoolGenius, RotoWire,
  FantasyLabs, My Survivor Pool for what that space already looks like — it's
  crowded and well-funded, not a good target to compete in directly).
- Keep it a single-file or minimal-build artifact unless there's a concrete
  reason to add a backend/build step.
