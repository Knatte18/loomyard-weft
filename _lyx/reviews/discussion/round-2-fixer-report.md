# Round 2 fixer report — discussion

## Changes

- **F1** (fixed as proposed, first option): D7 now says a row's time is wall time between history stamps, since the status file records no resume time (the step 3b resume write in `internal/shedengine/run.go` stamps nothing), so it includes every halt, pause and awaiting wait before the call ending the visit, and the wait before a `goto` entry is dropped.
  The checklist acceptance criterion pins a row resumed after a failed call and a row reached by `goto` after a halt.
- **F2** (fixed as proposed): D7's box vocabulary gains `halted`; the current producer's row shows running, failed, or halted for paused, blocked and awaiting (the six states in `internal/shedengine/status.go`), and its current visit's time keeps running to now in every state, consistent with F1's rule.
  The checklist acceptance criterion adds a blocked run.
- **F3** (fixed as proposed, second option): D8's append is once per batten process; a held-back or dropped notice composed again by the next batten process, or for a row notice by the next failure at the same history length, is logged again.
  The growth-bound sentence and the acceptance criterion's exactly-one-line wording now say per batten process, and the criterion pins the second line after a restart.
- Support log: a Round 2 entry under Review rounds.

## Disputed

None.

## Deferred

None.
