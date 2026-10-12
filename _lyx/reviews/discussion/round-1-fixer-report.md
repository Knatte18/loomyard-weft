# Round 1 fixer report — discussion

Every finding's premise held against the code (`internal/shedengine/run.go` `appendHistory` and `appendOrFold`, `goto.go`, `internal/orchcli/stop.go`, `internal/battenshed/notice.go` `deliver`, `internal/shedrun/seed.go`, `internal/reedengine/render/policy.go` and `rules.go`, commit bc9ee7a02 on `main`).
All eight are fixed as proposed.

## Changes

- F1: D7's time rule now builds on end times: a visit runs from the previous entry's `at` (the seed's `started_at` for the first) to its own `at`, and the running row from the last `at` to now.
  `repeats` count as visits with time running to `last_at`; a `goto` entry is no visit and starts the next one; a call with no recorded entry adds its time to the next visit.
  The checklist criterion gains a row with a folded repeat and a goto.
- F2: D3 states that when no sharing or positional strand is live, the window's bottom-most strand of any kind takes the free rows, a content strand then exceeding its maximum with blank rows.
  The orch criterion gains `lyx orch stop` leaving the `runs` strand and Selvage with every row assigned, and the render criterion a window whose only strand is a content strand.
- F3: D8 appends once per composed notice at its first delivery attempt, whether it is then queued, held back for a missing orch strand or dropped; the notice-log criterion gains a notice sent with no orch strand recorded.
- F4: D3 gains an outcome per anchor and kind pair (`top` with content allowed, `top` with positional or sharing refused by Rules, `below-parent` any kind, `hidden` ignored) and gives the even split's remainder rows to the bottom-most sharing strand; the render criterion covers both.
- F5: D2 shows a run with a seed and no batten status file as in flight, `starting`, and sorts a seed without `started_at` with the no-seed runs; D7 shows the state line and `recipe unreadable` when the seed or recipe cannot be read.
- F6: the Open risks bullet is removed; a Constraints bullet places the notice log outside every directory `lyx cleanup` sweeps and requires a rebase onto a `main` containing bc9ee7a02.
- F7: D2's departure paragraph is moved to the support log under the attach-command rejected alternative; A5 is cut to its pick, and its narrative moved to the support log's Interview item 5.
  The support log's lens-review lines that pointed at the moved text now point at its new place.
- F8: acceptance criteria added for D2's ordering and Worktree-Create rows, D1's `lyx orch stop`, and D7's removed-row entries, as new or extended rows of the existing overview, orch and checklist criteria.

The support log's Review rounds section records the round.

## Disputed

None.

## Deferred

None.
