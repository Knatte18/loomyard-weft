# Card 5 — poll-mode-backoff

**Create:**
- `plan:internal/reedengine#watchdogPollCeiling` -> `const watchdogPollCeiling = 60 * time.Second`
- `plan:internal/reedengine#pollWakeChanged` -> `func pollWakeChanged(res ReapplyResult, err error, lastApplied render.Box) bool`

**Edit:**
- `internal/reedengine#watchdogPollCycle`
- `internal/reedengine#watchModePoll`
- `internal/reedengine#watchTiming`
- `internal/reedengine#watchDefaultTiming`
- `internal/reedengine#Engine.watchLoop`
- `internal/reedengine/watchloop_test.go`
- `internal/reedengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

**Delete:**
- `internal/reedengine#tickerPeriodFor`

**Uses:**
- `internal/reedengine#NextWakeCadence`
- `internal/reedengine#oneShot`
- `internal/reedengine#Engine.handleWatchOutcome`

**Intent:**
Decision D5: the resize watch loop's poll mode wakes from a re-armed timer, not a fixed ticker.
`watchTiming` gains a `PollCeiling time.Duration` field; `watchDefaultTiming` fills it from the new `watchdogPollCeiling`, 60 seconds, the ceiling the discovery and name-repair loops already use.
`watchdogPollCycle` keeps its two-second value and its doc becomes "the poll-mode base cadence"; `watchModePoll`'s doc says it re-applies at a backed-off cadence.

`pollWakeChanged` is the pure rule for one poll tick, judged before `handleWatchOutcome` moves `lastApplied`: it answers true, which returns the wait to the base, when `err` is nil and the result is deferred, or applied a layout, or carries a live box that differs from `lastApplied`; it answers false, which doubles the wait, on a failed re-apply, on an unchanged live box with nothing applied, and on a degraded box with nothing applied.
A deferral returns to the base because another op is changing the session; a failure doubles so a broken tmux costs at most one round trip a minute; a degraded box is no observation.

`watchLoop` replaces its `time.Ticker` with a third `oneShot`, `wake`: poll mode arms it with `NextWakeCadence(wait, t.PollCycle, t.PollCeiling, changed)` after each tick, starting from the base at entry to the mode; dormant mode arms it with `t.Dormant` after each tick, keeping its fixed minute; signal mode arms nothing and stops it.
`tickerPeriodFor` goes with the ticker; `watchLoop` and `watchloop_test.go` were its only callers.
Dormant and signal modes are otherwise unchanged, and the mode switch re-arms the wake from the new mode's base.
The guard in `cmd/lyx/daemonwakeups_test.go` already lists `watchloop.go`, and no sub-second literal is added.

Docs: `internal/reedengine/doc.go`'s resize-loop sentence ("polls at a two-second cycle until the window-resized hook is seen installed") states the backoff from two seconds to a minute across ticks that change nothing and the return to two seconds on an applied layout, a changed box or a deferral, and the dormancy bullet's "ordinary two-second poll" becomes "ordinary poll".
`pattern/PATTERN-daemon-wakeups.md`'s "What counts" list names the resize watch loop's poll mode with its base and ceiling beside the watchdog daemon entry.
Bound, in `doc.go`: a resize while a poll-mode watcher is backed off heals at the next tick, up to a minute late; Windows is poll-only, and on POSIX a watcher stays in poll mode when its file watch failed at promotion or until a later attach or apply installs the hook.

Tests, all in `watchloop_test.go` with no new top-level function: `TestWatchState` replaces its `TickerPeriodForAnswersPerModeCadence` step with a `PollWakeChangedPerOutcome` step, a table of one row per outcome above, and its default-timing step gains the `PollCeiling` field and asserts the ceiling is at or above the base; `watchdogTestTiming` sets `PollCeiling` to a small multiple of its `PollCycle`.
`TestWatchLoop_PollModeByDefault`, whose scripted session never changes, asserts that the `list-panes` count over its window stays under the count a fixed cadence would reach, computed from the test timing's base and ceiling, so the backoff is observed on the loop itself, and its doc comment follows.
Every other loop test keeps passing because a changed or applied tick returns to the base.

**ImpactSummary:** The loop's poll cadence and one pure rule change inside `watchloop.go` and `watchdog.go`; the daemon in `reedcli`, the re-apply op and the hook install are untouched, and a poll-mode session costs at most one re-apply a minute instead of one every two seconds.
