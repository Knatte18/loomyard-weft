# Card 2 — shuttle: a gated run's waiting turn end is not an arrival

**Edit:**
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/waiting_test.go`

**Intent:**
In `pollEventsTick`, when the run is gated (`len(run.gate) > 0`) and the newest event is `EventWaiting`, return no outcome even when every output file exists.
An ungated run keeps today's order, where existing output files win over a waiting event.

Without this, the turn end on which Merriam spawns the gate fixer fork in the background (card 16) re-arrives at the gate, because `outcome.yaml` and `summary.md` are still on disk from the first arrival, and the gate verifies the fork's half-edited tree.
A waiting turn end on a gated run is also not an idle boundary for a pending entry: the next real turn end is.

The change lets a gated run keep polling past a turn end that would have finalized it.
Bound: the deferral holds only while the session is live.
The run deadline and the liveness checks keep running, and `classifyDeadlineExpiry` and the liveness finalize still classify Done from the output files and evaluate the gate one final time, so `master_timeout_min` (or the row's own timeout) bounds a fork that never finishes.

Update `pollEventsTick`'s doc comment to state the gated exception.

Tests in `waiting_test.go`:
a gated run whose output files exist and whose last event is `EventWaiting` does not evaluate the gate and does not finalize;
the next `EventStop` after it evaluates the gate exactly once;
an ungated run with output files and a last `EventWaiting` still classifies Done, as `TestPollEventsTick_WaitingWithOutputFilesIsDone` pins today.

**ImpactSummary:** every gated shuttle run (Plan-Write, Discussion-Write, Describe, PR-Rework, the gated Burler rows, and Merriam from card 15) stops finalizing on a waiting turn end; ungated runs are unchanged.
