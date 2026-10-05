# Card 3 — Inject the watchdog daemon's timing per command instance

**Edit:**
- `internal/reedcli#reedCLI`
- `internal/reedcli#reedCLI.watchdogCmd`
- `internal/reedcli/watchdog_integration_test.go`

**Uses:**
- `internal/reedcli#watchdogTiming`
- `internal/reedcli#watchdogDefaultTiming`

**Intent:**
The reed watchdog's discovery interval becomes a per-instance seam (D5).
`reedCLI` gains an unexported `watchdogTiming` override field;
`watchdogCmd` runs `runWatchdogLoop` with that override when it is set and with `watchdogDefaultTiming()` otherwise.
Production never sets the field, so the daemon keeps its 5 s discovery cycle, three idle cycles and three orphan-gone cycles;
the seam lets a test shorten only how long it waits, not what the daemon does.
It is a struct field, never a package-level `var`, so each test owns its own value and keeps `t.Parallel`.

In `watchdog_integration_test.go`, `runWatchdogCmdInBackground` builds its `reedCLI` with a compressed timing (sub-second discovery cycle, production cycle counts), and every test that drives the loop with `watchdogDefaultTiming()` or sleeps `watchdogHubDiscoveryCycle` switches to a compressed timing and waits in multiples of its own cycle instead:
`TestWatchdogIntegration_ExitsAfterIdleCyclesAndReleasesLock` (17.4 s in the measured run) and `TestWatchdogIntegration_ResizeAppliesOnlyToThatWorktree` (5.4 s) are the two the before-state table shows.
Their assertions stay as they are: the idle-exit test still proves the daemon exits after its idle cycles and releases the lock, now in a fraction of a second per cycle.
`watchdog_test.go`'s existing pin of `watchdogDefaultTiming` keeps guarding production's values.

**ImpactSummary:** reedcli only: one unexported field read in `watchdogCmd`, nil in production; test timing changes are local to `watchdog_integration_test.go`.
