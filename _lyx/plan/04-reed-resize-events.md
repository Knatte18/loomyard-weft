# Card 4 — reed-resize-events

**Create:**
- `plan:internal/reedengine#FileWatchOpener` -> `type FileWatchOpener func(dir string, names ...string) (<-chan fswatch.Event, func() error, error)`
- `plan:internal/reedengine#OpenFileWatch` -> `func OpenFileWatch(dir string, names ...string) (<-chan fswatch.Event, func() error, error)`

**Edit:**
- `internal/reedengine#watchTiming`
- `internal/reedengine#watchDefaultTiming`
- `internal/reedengine#tickerPeriodFor`
- `internal/reedengine#Engine.Watch`
- `internal/reedengine#Engine.watchLoop` -> `func (e *Engine) watchLoop(ctx context.Context, t watchTiming, open FileWatchOpener) error`
- `internal/reedengine#Engine.handleWatchOutcome`
- `internal/reedengine#watchdogDebounceQuiet`
- `internal/reedengine#watchdogRetryBaseDelay`
- `internal/reedengine/watchloop_test.go`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/fswatch#Watch`
- `plan:internal/fswatch#Event`

**Intent:** The resize loop's signal mode waits on the signal file through `fswatch` instead of a 100 ms stat tick (D4).
`FileWatchOpener` is the event-source seam the loop is handed: `OpenFileWatch` is the production value, wrapping `fswatch.Watch` on the told directory and returning the watcher's event channel and its `Close`, and a test passes an opener that returns a channel it feeds.
`Watch` calls `watchLoop` with `watchDefaultTiming()` and `OpenFileWatch`.
In signal mode the loop blocks on a select over the opener's events for `resizeSignalFileName`, a one-shot debounce timer armed from `watchState.Signal`'s quiet window, a one-shot retry timer armed from `watchState.Failed`'s escalating delay, and `ctx.Done()`; the ticker runs in poll and dormant mode only, and `tickerPeriodFor` loses its signal-mode case.
An event is handled as a stat tick was: stat the signal file, remove it when found, `Signal(now)`, and arm the debounce timer; an event that finds no file, such as the one the loop's own remove raises, runs nothing.
The timer's firing runs the apply through the existing `Plan` and `handleWatchOutcome` path, and a failed apply arms the retry timer from `readyAt`, up to `MaxAttempts` as today.
Promotion to signal mode, in `handleWatchOutcome`'s caller, opens the watcher on `e.stateDir()` for the signal file; when the opener fails the loop logs that once at Warn, stays in poll mode and never promotes again.
The watcher is closed when the loop returns.
`watchTiming.SignalTick` goes, which leaves `watchdogSignalTick` unread until the next card deletes it with the doc sentences that name it; `watchdogDebounceQuiet` and `watchdogRetryBaseDelay` carry `//lyx:one-shot` with their reasons (a debounce after an event, a bounded retry after a failed apply) on their declaration lines.
`watchloop_test.go` drives the loop with a fake opener: `TestWatchLoop_SignalMode` writes the signal file and sends one event per write, asserting no stat and no tmux call while no event arrives, one apply after a burst settles, and the file removed before the apply; new rows cover the retry on a failed apply with the escalating delay and abandonment after the cap, an event that finds no file running nothing, and an opener that fails leaving the loop in poll mode with one Warn and no later promotion.
A new test asserts `watchDefaultTiming`'s periodic fields (`PollCycle`, `Dormant`) are at or above one second, the D2 floor assertion for this package.
The existing `tmux`-tier self-heal tests keep running the production opener live.
`doc.go`'s watchdog section replaces the signal-tick sentences with the event wait and the poll fallback, and `cmd/lyx/testdata/test-budget.yaml` raises `internal/reedengine` as the new tests need.

**ImpactSummary:** The resize loop's signal mode changes its wait mechanism behind the same `Watch` surface, and the loop's unexported driver gains an opener parameter whose only callers are `Watch` and the loop's own tests.
