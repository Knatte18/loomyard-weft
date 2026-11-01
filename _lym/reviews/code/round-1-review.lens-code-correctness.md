```yaml
lens: code-correctness
outcome: reported
```

Task: code-correctness lens on the dead-driver grace and bootstrap-lock diff.

**Per function or branch, case run, verdict**

1. `noticeCondition` grace (`runnotice.go:250-253`), first tick that sees a new running child.
   - `runNotice` sets `memory.Since = child.Modified` (or `now` if that is zero) before the call (`runnotice.go:191-197`).
   - Data: Since = 06:18:50, now = 06:18:51, no sleeps, no pause. `AwakeSince` = 1s, which is below the minute, so the grace returns an empty finding. Correct.
   - `Since` is never zero here.
   - Zero `Modified` falls back to `now`, so the grace is a full minute. Correct.
   - A `Modified` in the future clamps to 0 in `AwakeSince`, so the notice is delayed. Safe.

2. Grace placement relative to the earlier returns.
   - A non-running child returns first. A covered pause returns before the liveness read. An unreadable `DriverAlive` returns before the grace.
   - So the grace only gates `!alive`. A live driver goes straight to `judgeAgents`, unaffected. Correct.
   - `pause.Intervals` is only read after the `Covered` return, so the exclusion list is the right one. Correct.

3. Grace return of `noticeFinding{}, activityStamp{}`.
   - It is identical to the old dead-driver return (`activityStamp{}` has `known=false`).
   - In `runNotice`, `keepEpisode` therefore keeps quiet, api-error and usage-limit keys. It keeps a driver-dead key only while its stamp equals `Since`.
   - `finding.condition == ""` returns early, and the deferred save still runs. The marker and memory behave as on any no-condition tick. Correct.

4. Stale `Since` (LOW, `design`).
   - `Since` only resets when `ChildState` changes or `Since` is zero (`runnotice.go:191`).
   - If a halt and resume falls between two probe ticks, dobby never sees a non-running state. `Since` stays at the old episode, so the grace has long expired.
   - The summary claims the grace applies "whenever a child enters running, such as a resume after a halt" (summary.md, Design decisions). That holds only if dobby saw the intermediate state.
   - The bootstrap lock still covers most of that window. This is a residual, narrow false-notice path.
   - Evidence: `runnotice.go:190-197`.

5. `childDriverLockHeld` (`dobbynotice.go:131`), bootstrap lock path.
   - `loomengine.LoomBootstrapLock(taskLocation)` is the same function `lym loom start` uses with `c.location` (`start.go:533`), and `taskLocation` is the task worktree. The path matches.
   - A missing lock file: `runLockHeld` runs `MkdirAll`, then `TryAcquireWriteLock`. The lock is free, so it is released and the function returns `held=false`. Correct, and the `NoLockHeld` row covers it.
   - A held lock returns `held=true`. An unreadable lock returns `true` and is logged. Correct.
   - The loop short-circuits on the first held lock.
   - `loomengine` is already imported by other `battencli` files (`wire.go`, `drive.go` and others), so there is no new layering edge.

6. Probe contention with `lym loom start` (not a defect).
   - `lym loom start` takes the bootstrap lock with the blocking `AcquireWriteLock` (`lock.go:30`).
   - A concurrent dobby probe that holds the lock for an instant only delays the start for microseconds. It does not fail it.
   - The probe path is the same one used for the driver and run locks.

7. Doc and comment accuracy.
   - The `childDriverLockHeld` comment says the bootstrap lock is held "from before it spawns the driver until the driver holds its own lock". This matches `start.go:538-593` (released after the handshake).
   - `lym loom start` writes the status before it takes the lock (the summary's own point), so the grace is what covers that gap. The two fixes together are correct.

**Findings**

- **LOW, `design`:** the stale `memory.Since` can defeat the grace when dobby misses the non-running state. Location: `internal/dobbyengine/runnotice.go:191-197`, with the claim in `_lym/landing/summary.md`. Fix direction: either reword the summary so the grace is not claimed for every re-entry to running, or key the episode on something that changes on re-entry.

No other correctness defects found.

**Tool calls**
1. `grep` of `internal/lock/lock.go` for `AcquireWriteLock` and `TryAcquireWriteLock`: my context lacked whether the start side blocks or fails when a probe holds the lock.
