# Card 3 — shuttle-shell-expiry

**Edit:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Run.pollEventsTick`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine/waiting_test.go`

**Uses:**
- `plan:internal/shuttleengine#BackgroundTask`
- `plan:internal/shuttleengine/claudeengine#outstandingBackgroundTasks`
- `internal/shuttleengine#Config`

**Intent:**
D1's wait-loop half and D2's shuttle half: a turn end waiting only on background shells counts as a turn end once each shell has been outstanding longer than `background_shell_wait_min`.

`Spec` gains `AwaitedShellPrefixes []string`: a shell whose label starts with one of them is waited on like a fork, bounded only by the run's own timeout.
It is caller data, not provider knowledge; `Spec.validate` does not inspect it.

`Run` gains the state the expiry needs across ticks: the outstanding tasks of the latest `EventWaiting` (cleared by any later non-waiting event), when each shell id was first seen outstanding, and the ids already expired in this shuttle run with their labels in expiry order.

`pollEventsTick`:
- On a batch whose last event is `EventWaiting`, record its `Outstanding` list and stamp each shell id not yet seen with the clock's now.
- On every tick, with or without new bytes, while a waiting list is recorded: when every outstanding task is a shell that is not awaited and each is either already expired or outstanding at least the bound, mark each newly expired id expired, log each through `internal/logger` (Warn, with the run dir and the shell label), clear the waiting list, and classify the tick as a turn end:
  `OutcomeDone` when every output file exists, otherwise `OutcomeAsking` with the waiting event's message, exactly as a Stop would classify.
  A gated run reaches `handleGatedBoundary` through the existing Done branch in `Wait`, so the expiry is an arrival.
- A fork, or an awaited shell, in the list keeps the turn waiting, as today.
- An id already expired stays expired for the rest of this run, so a later Stop that lists it again does not restart the wait.
The check reads the bound from `Config.BackgroundShellWaitMin`;
a non-positive value in a hand-built `Config` falls back to the template default (10), mirroring `pollInterval`'s floor, since `LoadConfig` already refuses one from a file.
The expiry's negative answer (`OutcomeAsking`) is reached only after `allOutputFilesExist` is consulted (PATTERN-completion-signal).

`Result` gains `ExpiredShells []string`, the labels of the shells this run waited out, in expiry order; `finalize` fills it.

Bound: the expiry lets a turn end through while non-awaited shells still run, at the earliest `background_shell_wait_min` after each was first seen;
it never lets one through while a fork, or an awaited shell within the run timeout, is outstanding.

Tests (fake clock, no new events after the waiting Stop):
a waiting turn end with one non-awaited shell becomes Done after the bound and stays waiting just before it;
the same with output files missing classifies asking;
a fork outstanding stays waiting past the bound;
a shell whose label starts with an awaited prefix (`lyx webster recover-batch 03`) stays waiting past the bound;
a later Stop listing an expired id again ends the turn on that tick without a new wait;
a gated run's expiry evaluates the gate;
`Result.ExpiredShells` names the label.
Document the bound, the awaited prefixes and `ExpiredShells` in `internal/shuttleengine/doc.go`.

**ImpactSummary:** the wait loop gains one expiry branch reached only from a waiting turn end; every other classification path is unchanged.
