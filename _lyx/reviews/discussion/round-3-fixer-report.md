# Round 3 fixer-report — discussion

All six findings were validated against the code and fixed.

## Fixed

- **F1 (BLOCKING, Linux step descendants).**
  Confirmed: `internal/gatecli/child_linux.go` sets `Pdeathsig`, which reaches the direct child only, and `proc.StartTime` reads `/proc/<pid>/stat`, so a start-time gate on the dead leader skips the group kill.
  D2 now states that each child runs in its own process group and defines the tree kill: SIGKILL to the recorded group, gated on a live process reporting that group with a start time at or after the recorded child start, which holds after the leader is dead.
  Descendants that leave the group are reached only by their own parent-death tie.
  The dead-loop stop runs the tree kill first.
  D11 kills the loop, then the group under that gate.
  The teardown and loop-killed acceptance rows now assert that the child's descendants die.
- **F2 (BLOCKING, Windows start time).**
  Confirmed: `internal/proc/proc_windows.go`'s `StartTime` returns `"", false`.
  `proc.StartTime` is left unchanged because the claude engine's resume check relies on it failing closed.
  D2 instead gives the loop a named kill-on-close Job Object, named from the steps directory path, that the loop joins at start and that nests the per-child jobs.
  The name exists only while the loop holds its handle, so no reused pid can match.
  D11 terminates that job on Windows.
  The teardown acceptance row names both platforms.
- **F3 (MEDIUM, Windows detach).**
  Confirmed: on Windows, `Detach` sets no breakaway flag and `DetachBreakaway` adds `CREATE_BREAKAWAY_FROM_JOB`.
  D2 now spawns with `DetachBreakaway`.
  When the job forbids breakaway, it falls back to `Detach` and logs the fallback; the remaining risk is listed under Open risks.
  The `lyx loom start` precedent citation is removed.
- **F4 (MEDIUM, envelope currency).**
  Confirmed: the step-3b resume write in `internal/shedengine/run.go` changes neither the producer nor the history.
  D1's `loop` object gains `state`, D2's currency key now includes it, and the "moved past by a resume" premise is restated in terms of `state`.
  D2 states that a waiter retires the envelope it prints.
  The fold case (`appendOrFold`) is listed under Open risks.
  The acceptance row covers retire-on-print and a halt envelope after a resume.
- **F5 (MEDIUM, busy dead-loop stop).**
  D2 states that a dead-loop stop meeting another run-lock holder ends `busy`, keeps the pid file, writes no loop envelope, and is retried by the next invocation.
  The loop-killed acceptance row covers this case.
- **F6 (LOW, lock-race loser).**
  D2 defines "gone" by the loop lock alone: the lock is free and no envelope exists.
  A waiter whose loop exited while the lock is held waits on the holder.
  Line 70's parenthetical is removed.
  The acceptance row asserts that the loser's waiter writes no `failed`.

The support log's question ledger records the round-3 picks (tree kill and Windows identity, envelope currency).

## Disputed

None.

## Deferred

None.
