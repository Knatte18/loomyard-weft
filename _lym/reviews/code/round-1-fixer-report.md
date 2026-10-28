```yaml
round: 1
fixed: [F1, F2, F3, F4, F5, F6, F7, F8]
disputed: []
deferred: []
```

## Changes

Each finding is one commit, `git log` on the worktree.

- F1 fixed as proposed: `runLoop` takes the status file's producer for an interrupted stop only when it is non-empty (`internal/shedverbs/loop.go`).
  The regression test is the F5 subtest "child dies leaving the status without a producer".
- F2 fixed as proposed: `ClassifyHalt` reports `interrupted-stop` for any non-blocked state of an interrupted halt, and the `AnomalyInterrupted` and `ClassifyHalt` doc comments say so (`internal/loomengine/anomaly.go`).
  The `RunningInterrupted_None` row asserted the narrower rule, so it became `RunningInterrupted_InterruptedStop`, and a `DoneInterrupted_InterruptedStop` row was added.
- F3 fixed as proposed: `internal/gitexec` gained the `sleepBeforeRetry` variable, `SetStartFailuresForTest` records the waits instead of sleeping and returns them, and `TestRun_StartRefusedForEAGAIN` asserts the waits double from a non-zero first delay and gained a `RemoteRefusedEveryTry` row over `ls-remote`.
- F4 fixed as proposed: `TestSpawnRefusedRecently` in `internal/shedtransient/shedtransient_test.go`, with rows inside, at and beyond `spawnRefusalWindow`.
  The test is in the package's own test package, so it reaches `spawnRefusedRecently` directly and needs no `export_test.go`.
- F5 fixed as proposed: four subtests in `TestLoop_InterruptedChildFillsTheErrorForm`, for a status without a producer, a status file unreadable after the child exited, an empty stderr with no trace file, and nil `StopFiles`.
- F6 fixed as proposed: a `TestUntilStop_DeadLoopStopsLoopExited` subtest over a paused status asserts `loop.pause_reason`.
- F7 fixed as proposed: a `NoStatusFile_FallsBackToFailed` row in `TestLoomAfterInterrupt_WritesTheHaltNoteAndNoReflection`.
- F8 fixed as proposed: a `TestParseDriveStop` row with a non-paused loop state and a set `pause_reason`, asserting `Stop.Reason` stays empty.

## Disputed

None.

## Deferred

None.

## Self-check

Four forks returned a report, each with no findings: fix-code-correctness, fix-consistency, fix-proposal, fix-affected-set.
No fork returned empty.
