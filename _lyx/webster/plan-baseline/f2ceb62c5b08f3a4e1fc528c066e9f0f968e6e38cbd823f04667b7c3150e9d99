# Card 20 — watchdog-kill-record

**Create:**
- `internal/shedverbs#WatchdogKill`
- `internal/shedverbs#WatchdogKill.cause`
- `internal/loomcli#watchdogKillNote`
- `internal/loomcli#loomCLI.stopStepRuns`
- `internal/loomcli#TestWatchdogKillNote_StopsTheKilledStepsRunsAndNamesTheRest`

**Edit:**
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#watchIdle` -> `func watchIdle(ctx context.Context, w idleWatch) (WatchdogKill, bool)`
- `internal/shedverbs#waitForChild` -> `func waitForChild(ctx context.Context, loop LoopSpec, child Child, started time.Time, traceID string, window time.Duration, waitDone <-chan error) (*WatchdogKill, error)`
- `internal/shedverbs#killIdleChild` -> `func killIdleChild(ctx context.Context, sleep func(context.Context, time.Duration) error, child Child, waitDone <-chan error, kill WatchdogKill) error`
- `internal/shedverbs#runChild`
- `internal/shedverbs#TestLoop_WatchdogKillsAnIdleChild`
- `internal/loomcli#loomCLI.specFor`
- `internal/loomcli#loomCLI.buildLoomShed`
- `internal/shedverbs/doc.go`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Issue #567 (D10): a watchdog kill names itself in the halt note and the trace, and the killed step's shuttle runs get a terminal record, so the step's relaunch starts its agent fresh instead of reattaching to a strand that lost its task.

shedverbs:
- `WatchdogKill` carries `TraceID`, `Window`, `Idle` (the awake, unpaused idle time card 19 measures) and `LastActivity` (the newest activity time the watch read, the child's start when none).
  `watchIdle` returns it, `waitForChild` returns a pointer to it for a killed child and nil otherwise, and `killIdleChild` logs `window`, `last_activity` and `idle` on both of its Warn lines, so the step's trace names the watchdog's reading.
- `LoopSpec` gains `OnWatchdogKill func(WatchdogKill) string`, nil skipped; `runChild` calls it once after a kill, before it reads back what the child left.
- The interrupted cause of a killed child becomes `kill.cause(note)`: `watchdog: the step watchdog killed the step after <idle> of awake idle time (window <window>, last activity <RFC 3339>)`, followed by `; <note>` when the hook returned one.
  The loop's stop note keeps naming the watchdog, and the cause reaches the status file's error, the envelope's `error`, and the halt note's reason through `AfterInterrupt`, which loom writes as the reflected halt note.

loomcli (the arming module, not the generic `shedverbs`):
- `buildLoomShed`, the step verb's shed builder, tells `c.runner` the step tie `shuttleengine.StepTie{StepsDir: shedrun.StepsDir(c.location, c.runID), TraceID: logger.TraceID()}` (card 10); a loop's step child adopts its trace id from `LYX_TRACE_ID`, so the tie names the step the loop runs. `lyx loom start` never builds a step shed, so the driver's strand carries no step.
- `stopStepRuns` stops through `c.runner`, building the pair's runner the way `wire` does when the receiver was wired lightly.
- `watchdogKillNote(stop, stepsDir)` returns the hook `specFor` sets on `spec.Loop.OnWatchdogKill`: it calls `stop` with the killed step's tie and renders `stopped shuttle runs: <names>` and, for live runs whose record names no step (an older binary's), `left live runs naming no step: <names>`; a stop error is rendered as `could not stop the step's runs: <error>`.
Bound: the stop takes only runs whose record names the killed step; the loom driver's strand, the orch and another step's runs are left live (card 10).

`refusal-spec.md`'s interrupted-step row names the watchdog cause's fields.

Tests: `TestLoop_WatchdogKillsAnIdleChild` asserts the new error text with the window, the idle time and the fake clock's start as last activity, that an `OnWatchdogKill` hook is called once with the child's trace id and its note ends the error, and that the hook is not called for a child that exits on its own.
`TestWatchdogKillNote_StopsTheKilledStepsRunsAndNamesTheRest` drives the hook over a fake `stop` that records its tie and returns stopped and untied names: the tie is the steps directory and the kill's trace id, the note names both lists, and a failing stop renders its error.
The loomcli budget row grows by one.

**ImpactSummary:** shedverbs' watchdog functions re-signed with their only callers inside loopwatchdog.go and loop.go on this card; LoopSpec gains one optional hook that loom fills, and the interrupted cause text of a watchdog kill grows.
