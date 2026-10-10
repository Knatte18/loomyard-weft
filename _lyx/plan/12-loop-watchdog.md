# Card 12 — loop-watchdog

**Create:**
- `plan:internal/shedverbs#idleWatch` -> `type idleWatch struct`
- `plan:internal/shedverbs#watchIdle` -> `func watchIdle(ctx context.Context, w idleWatch) (time.Duration, bool)`
- `plan:internal/shedverbs#newestActivity` -> `func newestActivity(traceDir, traceID string, agent func() (time.Time, bool, error)) (time.Time, bool)`

**Edit:**
- `plan:internal/shedverbs#runLoop`
- `plan:internal/shedverbs#LoopSpec`
- `internal/shedverbs/loop_test.go`
- `internal/shedverbs/doc.go`
- `cmd/lyx#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/logger#TraceDir`
- `plan:internal/shedverbs#Child`

**Intent:** D4: the loop kills a child step that shows no activity for the idle window and reports it as an `interrupted` stop with cause `watchdog`.
`LoopSpec.IdleTimeout` is the window, told by the arming module (card 14), and `LoopSpec.Activity func() (time.Time, bool, error)` is the arming module's reading of the run's agent activity, newest over every live strand (a MultiLLM row's chair and advisors included), with a live verify wait marker counting as now; a zero `IdleTimeout` disarms the watchdog, and a window below one minute is floored to one minute with one Warn naming it.
`newestActivity` takes the newest of the child's trace group's mtime under `logger.TraceDir()` and the `Activity` hook's reading, and reports false when neither answers.
`idleWatch` carries the child, the window, the activity reading, a `Now` and a `Sleep`; `watchIdle` wakes at most once a second, doubling from one second to a one-minute ceiling while the reading is unchanged and returning to one second on a change, and returns the idle time and true once the window passes with no activity, or false when the context ends because the child exited.
When the window passes, `runLoop` calls `Child.Quit` so a Go child's goroutine dump lands in the captured stderr file, then `Child.Kill`, which takes the child with its descendants, and classifies the stop as `interrupted` with `loop.detail` naming `watchdog` and the idle time; the loop never re-invokes an interrupted step itself, and `loop.interrupt_policy` and `loop.status_moved` let the driver's rule decide.
A child that exits on its own with a non-zero status and no record keeps cause `exited` with the exit status.
`cmd/lyx#unboundedWaitFiles` lists `internal/shedverbs/loop.go`, and `PATTERN-daemon-wakeups.md` lists the step watchdog beside the waiter.
`doc.go` describes the watchdog and what counts as activity.
`loop_test.go` gains `TestLoop_WatchdogKillsAnIdleChild`: with a fake clock and a fake child that never exits, the stop is `interrupted` with `kind: interrupted`, cause `watchdog`, the row's policy, the status file written `failed` and the stderr path named; a row where the activity reading advances keeps the child alive past the window; a row with a zero window never kills.
`test-budget.yaml` raises `internal/shedverbs` from 80 to 81.

**ImpactSummary:** Internal to the loop body and its spec; a zero `IdleTimeout` leaves the loop of card 11 unchanged.
