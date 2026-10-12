# Round 2 fixer report — discussion

## Changes

- F1 (BLOCKING), fixed as proposed: D6 extends the watchdog-quit halt reason and the `busy_quit_sec` way forward to the loom verify gate (`internal/loomshed`) and the landing verify gate (`internal/landingshed`), and their refusal-spec rows (`Webster-Burler-Fix verify gate exhausted`, `Darn-Write verify gate exhausted`, the landing failed-verify rows) gain it in the same commit.
  Scope In gains lines for both packages, and their package docs join the docs scope line and the docs criterion.
  A new acceptance criterion pins the halt reason for each gate.
  D6 notes these gates run no flake rerun, so D5 does not change them.
- F2, fixed as proposed: D7's variable carries the enclosing `lyx gate test`'s pid, and the skip applies only when that pid is a live ancestor, walked through the parent pids `internal/proc` reads.
  D7's Bound says a variable naming no live ancestor is ignored, and the watchdog criterion adds that a non-ancestor pid still runs the watchdog.
- F3, fixed as proposed: the integration criterion strips D7's variable from the inner run's environment and asserts the watchdog line carries the 1-second window.
- F4, fixed as proposed: the injected-sampler criterion gains a `.test` process at 92 percent of one core being quit; the integration criterion spins at least two goroutines; D7's Rationale says the 10 percent margin covers tick granularity only.
  The threshold sentence in D7 drops slot neighbours as a cause the margin covers, and Open risks names a spinner starved by contention.
- Support log: round 2 entries added under Review rounds.

## Disputed

None.

## Deferred

None.
