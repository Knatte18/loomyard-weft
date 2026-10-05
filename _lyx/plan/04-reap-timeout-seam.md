# Card 4 — Inject the reap timeout into the watchdog's session reap

**Edit:**
- `internal/reedengine#ReapSession`
- `internal/reedengine#reapSessionVia`
- `internal/reedcli#watchdogTiming`
- `internal/reedcli#watchdogDefaultTiming`
- `internal/reedcli#dispatchReap`
- `internal/reedcli#runWatchdogLoop`
- `internal/reedcli/watchdogreap_integration_test.go`
- `internal/reedcli/watchdog_test.go`

**Uses:**
- `internal/reedengine#reapExitTimeout`
- `internal/reedengine#reapPaneChildren`

**Intent:**
`reapExitTimeout` becomes injectable on the path a test waits it out (D5).
`ReapSession` gains a trailing `timeout time.Duration` parameter and passes it through `reapSessionVia` to `reapPaneChildren`;
a zero timeout selects `reapExitTimeout`, so the constant stays unexported and remains the only production value.
`watchdogTiming` gains a `ReapTimeout time.Duration` field, `watchdogDefaultTiming` leaves it zero, and `runWatchdogLoop` hands its timing's `ReapTimeout` to `dispatchReap`, which passes it to `ReapSession`.
The doc comments in `watchdog.go` that quote the 15 s reap budget say it is that field's default.

`watchdogreap_integration_test.go`'s `compressedReapTiming` sets a short `ReapTimeout`, and `TestWatchdogReap_DescendantClosureConfirmedExited` (15.9 s in the measured run) bounds its `waitProcessGone` waits by that timeout plus the force-kill grace instead of a fixed 25 s.
It still proves a SIGHUP-resistant descendant is force-killed, now without waiting out the production value.
`watchdog_test.go`'s default-timing pin gains the zero `ReapTimeout` assertion.

The `Engine` methods that call `reapPaneChildren` (`Down`, strand removal, replace) keep `reapExitTimeout`: no measured test waits on them, so they get no seam in this card.
Bound: the parameter changes only how long the reap waits before force-killing;
a zero value, the only one production passes, is today's behavior.

**ImpactSummary:** `ReapSession` has one caller (`dispatchReap` in reedcli); `watchdogTiming` is unexported in reedcli; no other module sees the change.
