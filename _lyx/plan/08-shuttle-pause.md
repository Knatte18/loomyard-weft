# Card 8 — shuttle-pause

**Create:**
- `internal/shuttleengine#AwakeClock`
- `internal/shuttleengine#realClock.Awake`
- `internal/shuttleengine#Runner.SetPause`
- `internal/shuttleengine#Runner.PauseGeometry`
- `internal/shuttleengine#Run.observePause`
- `internal/shuttleengine#TestRunner_Start_WaitsWhileAPauseCoversItsPair`
- `internal/shuttleengine#TestRun_Wait_DeadlineExcludesSleptAndPausedTime`

**Edit:**
- `internal/shuttleengine#Runner`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Runner.reconstruct`
- `internal/shuttleengine#Run.Wait`
- `internal/shuttleengine#Run.notifyHeld`
- `internal/shuttleengine#Run.watchDialog`
- `internal/shuttleengine#TestWait_HeldTurnEndNotice`
- `internal/shuttleengine#TestWait_DialogNotice`
- `internal/shuttleengine#fakeClock`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Make shuttle the hold point for every agent spawn and a reader of the pause (D4, D5, D9): a run start waits while a pause covers its pair, Wait sends no hold or dialog notice for a covered run, and the run deadline counts awake, unpaused time.

Seam: `Runner.SetPause` tells the runner its `pauserecord.Geometry`; a runner never told one (the orch's, every test runner) reads no record and behaves as today. `PauseGeometry` returns it, for the wiring test of card 12.
`AwakeClock` is the optional `Awake() time.Duration` a run's `Clock` may also implement; `realClock` implements it through a package-level origin taken at first use, and a clock without it detects no sleep.

Start: `Runner.start`, after the spec and resume checks and before it creates the run directory, calls `pauserecord.WaitUncovered` for the runner's slug when a pause geometry is set; its one line goes to stderr and to the trace through `logger.Info`.
Every producer, advisor, Burler round and burler relaunch starts through here, so each waits at entry; the run's deadline is taken after the wait, so a started run's deadline counts from its start.
Bound: the start only waits, with no deadline of its own, and starts nothing while held.

Wait: each tick calls `observePause` before the deadline check.
It feeds `run.sleeps` (a `pauserecord.SleepWatch`, new on `Run`) with the clock's wall and awake readings when the clock is an `AwakeClock`, reads the runner's coverage, sets `run.covered`, and moves `run.deadline` to `run.baseDeadline` (the deadline `start` or `reconstruct` sets, kept beside it) plus `pauserecord.Excluded` over the detected sleeps and the coverage's paused intervals since the run was created.
While `run.covered` holds, the deadline check is skipped, `notifyHeld` returns without sending or advancing `NotifiedOffset`, and `watchDialog` latches but sends no notice; a held turn end read while covered is not re-notified after the resume, since the resume line starts the next turn.
An unreadable record reads uncovered with one Warn per change of error.

Tests:
- `TestWait_HeldTurnEndNotice` gains a row whose runner is told a geometry over a temp record with a live entry for its slug: the held stop sends no notice and leaves `NotifiedOffset` at zero.
- `TestWait_DialogNotice` gains a row under the same live entry: a dialog latched past `permissionNoticeBound` sends no notice.
- `TestRunner_Start_WaitsWhileAPauseCoversItsPair`: with a live hub entry and a `fakeClock`, `Start` in a goroutine has called no `AddStrand` on the fake reed after the wait's first sleep; the test resumes the entry, and `Start` then returns a run whose deadline is `Timeout` past the resume, not past the call.
- `TestRun_Wait_DeadlineExcludesSleptAndPausedTime`: `fakeClock` gains an `awake` reading and an `asleep` jump that moves wall time alone; a run with a one-hour timeout across a three-hour sleep is still running past its wall deadline and times out one awake hour after its start; a second row covers the run for three hours, during which no tick times it out, and it times out one unpaused hour after its start.
`doc.go` gains the pause and awake-deadline paragraphs; the budget row grows by two.

**ImpactSummary:** Shuttle's Runner and Run gain a told pause geometry and a sleep watch; runners never told one are unchanged, and the four held paths are start, deadline, hold notice and dialog notice.
