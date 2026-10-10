# Card 5 — poll-mode-backoff

**Create:**
- `internal/reedengine#watchdogPollCeiling`
- `internal/reedengine#pollWakeChanged`

**Edit:**
- `internal/reedengine#watchdogPollCycle`
- `internal/reedengine#watchModePoll`
- `internal/reedengine#watchTiming`
- `internal/reedengine#watchDefaultTiming`
- `internal/reedengine#Engine.watchLoop`
- `internal/reedengine/watchloop_test.go`
- `internal/reedengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

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
The ticker was `tickerPeriodFor`'s only production caller; card 6 deletes the function and its test step, and this card leaves both in place.
Dormant and signal modes are otherwise unchanged, and the mode switch re-arms the wake from the new mode's base.
The guard in `cmd/lyx/daemonwakeups_test.go` already lists `watchloop.go`, and no sub-second literal is added.

Docs: `internal/reedengine/doc.go`'s resize-loop sentence ("polls at a two-second cycle until the window-resized hook is seen installed") states the backoff from two seconds to a minute across ticks that change nothing and the return to two seconds on an applied layout, a changed box or a deferral, and the dormancy bullet's "ordinary two-second poll" becomes "ordinary poll".
`pattern/PATTERN-daemon-wakeups.md`'s "What counts" list names the resize watch loop's poll mode with its base and ceiling beside the watchdog daemon entry.
Bound, in `doc.go`: a resize while a poll-mode watcher is backed off heals at the next tick, up to a minute late; Windows is poll-only, and on POSIX a watcher stays in poll mode when its file watch failed at promotion or until a later attach or apply installs the hook.

Tests, all in `watchloop_test.go` with no new top-level function: `TestWatchState` gains a `PollWakeChangedPerOutcome` step, a table of one row per outcome above, and its default-timing step gains the `PollCeiling` field and asserts the ceiling is at or above the base.
`TestWatchLoop_PollModeByDefault`, whose scripted session never changes, observes the backoff on the loop itself: it runs on its own copy of the test timing with `PollCeiling` at `8 * PollCycle`, 40 ms, and after its existing three-tick wait it sleeps a 200 ms window and asserts the window's `list-panes` growth is at most twelve ticks' worth, a tick's worth being the `list-panes` calls one re-apply issues on the fixture's session, written as a named constant.
The backed-off schedule fits about six ticks in that window and a fixed 5 ms cadence about forty, so the bound fails a regression to a fixed cadence by a wide margin and scheduling latency only lowers the backed-off count; the test's doc comment follows.

A sweep of `watchloop_test.go` found every loop test that stays in poll mode on ticks that double the wait, each with the timing windows it needs:
`TestWatchLoop_PollModeByDefault`, three ticks within 300 ms before the window above;
`TestWatchLoop_TakeEffectBoundary`, two ticks within 200 ms, then one more within 200 ms;
`TestWatchLoop_OpenerFailureStaysInPollMode`, on an unchanged session, and `TestWatchLoop_NonSentinelFailureDoesNotGoDormant`, on failing re-applies, five ticks within 300 ms each;
and `TestWatchLoop_PollModeGoesDormantOnVanishedWorktreeRoot`, two ticks within 200 ms, then a 20 ms window in which `list-panes` must stop growing because the loop went dormant, never because it is waiting out a backed-off tick.
`TestWatchLoop_UndecidedProbeDoesNotGuess` defers, which returns to the base, and the signal-mode tests promote on their first tick, so `waitForPromotion`'s 20 ms windows run with no wake armed.
So `watchdogTestTiming` sets `PollCeiling: 2 * PollCycle`, 10 ms, with a comment naming both bounds: the base plus four ceiling waits, 45 ms, fit well inside the 300 ms windows, and the ceiling stays strictly below the dormancy test's 20 ms stability window, so a still-polling loop always shows a tick inside it.
`TestWatchLoop_OpenerFailureStaysInPollMode`'s and `TestWatchLoop_NonSentinelFailureDoesNotGoDormant`'s doc comments, which say the loop keeps polling at the poll cadence, say it keeps polling at the backed-off poll cadence.

**ImpactSummary:** The loop's poll cadence and one pure rule change inside `watchloop.go` and `watchdog.go`; the daemon in `reedcli`, the re-apply op and the hook install are untouched, and a poll-mode session costs at most one re-apply a minute instead of one every two seconds.
