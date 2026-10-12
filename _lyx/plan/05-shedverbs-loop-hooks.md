# Card 5 — shedverbs-loop-hooks

**Create:**
- `plan:internal/shedverbs#TestLoop_ExitHookRunsOnceAtEveryStop` -> `func TestLoop_ExitHookRunsOnceAtEveryStop(t *testing.T)`
- `plan:internal/shedverbs#TestLoop_WakeHookRunsOnWatchWakes` -> `func TestLoop_WakeHookRunsOnWatchWakes(t *testing.T)`

**Edit:**
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#idleWatch`
- `internal/shedverbs#watchIdle`
- `internal/shedverbs#waitForChild`
- `internal/shedverbs/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Give the detached loop two told seams the arming module fills (D5, D7), keeping `shedverbs`' import set unchanged (`PATTERN-shed-verb-set`).

`LoopSpec.OnExit func(stop LoopStop, st shedengine.Status)` runs once in `runLoop` after the stop is final, that is after a failed stop is written, with the status file as it then reads, and before the envelope is written.
It runs at every stop kind (halted for awaiting, blocked, done or paused, stop condition, error, interrupted and busy), so the arming module can release whatever the loop process holds.
A nil `OnExit` is skipped.
A loop that refuses before its first step, and a dead loop the waiter reports, never run it: a dead loop's records lapse by its dead pid.

`LoopSpec.OnWake func()` runs on every wake of the child watch.
`idleWatch` gains a `Wake func()` field that `watchIdle` calls on each wake before it judges the idle time.
`waitForChild` passes `OnWake` through, and when the watchdog window is zero but `OnWake` is set, it runs the same watch with no kill, waking from one second doubling to a minute (`PATTERN-daemon-wakeups`), until the child exits.
A nil `OnWake` keeps today's behaviour.

`TestLoop_ExitHookRunsOnceAtEveryStop` drives the fixture loop to a halted, an error and an interrupted stop and asserts the hook ran exactly once each, with that stop and the final status.
`TestLoop_WakeHookRunsOnWatchWakes` runs a child with the watchdog disarmed and the fake clock and sleep, and asserts the hook ran on the watch's wakes and stopped when the child exited.
`doc.go` names both seams beside the watchdog.

**ImpactSummary:** two optional loop seams; with both nil every loop behaves exactly as before.

**Commit:** 5: shedverbs: loop exit and wake hooks
