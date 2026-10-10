# Card 6 — watchdog-discovery-backoff

**Create:**
- `plan:internal/reedcli#watchdogHubDiscoveryCeiling` -> `const watchdogHubDiscoveryCeiling = 60 * time.Second`
- `plan:internal/reedcli#discoveryWake` -> `type discoveryWake struct`
- `plan:internal/reedcli#discoveryWake.afterCycle` -> `func (d *discoveryWake) afterCycle(now time.Time, changed bool)`
- `plan:internal/reedcli#discoveryWake.onSignal` -> `func (d *discoveryWake) onSignal(now time.Time)`
- `plan:internal/reedcli#discoveryWake.next` -> `func (d *discoveryWake) next(now time.Time) time.Duration`
- `plan:internal/reedcli#sameSessionSet` -> `func sameSessionSet(a, b []string) bool`

**Edit:**
- `internal/reedcli#watchdogHubDiscoveryCycle`
- `internal/reedcli#watchdogTiming`
- `internal/reedcli#watchdogDefaultTiming`
- `internal/reedcli#runWatchdogLoop` -> `func runWatchdogLoop(ctx context.Context, hub, tmuxPath, shellPath string, timing watchdogTiming, open reedengine.FileWatchOpener) error`
- `internal/reedcli#reedCLI.watchdogCmd`
- `internal/reedcli/watchdog_test.go`
- `internal/reedcli/watchdog_integration_test.go`
- `internal/reedcli/watchdogreap_integration_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/reedengine#NextWakeCadence`
- `plan:internal/reedengine#DiscoverSignalFileName`
- `plan:internal/reedengine#OpenFileWatch`

**Intent:** The discovery loop backs off while the session set is unchanged and wakes on the discover signal (D5, the `reedcli` half).
`watchdogTiming` gains `DiscoveryCeiling`, filled from `watchdogHubDiscoveryCeiling` by `watchdogDefaultTiming`; `watchdogHubDiscoveryCycle` stays the 5 s base and its comment names the backoff.
`discoveryWake` is the pure wake planner: it holds the last cycle's time and the current cadence; `afterCycle` records a cycle and moves the cadence with `reedengine.NextWakeCadence` (reset to the base when the set changed, doubling to the ceiling otherwise); `onSignal` marks a consumed signal pending; `next` returns how long to wait from `now`, the time until the pending signal's cycle, which is never sooner than the base after the last cycle, else until the cadence ends.
`sameSessionSet` compares two listings as sets.
`runWatchdogLoop` replaces its ticker with a timer re-armed from `next` after every wake, opens the hub-level discover signal through `open` on `fabricengine.HubScratchDir(hub)` for `reedengine.DiscoverSignalFileName`, and selects over the timer, the events and `ctx.Done()`.
An event wake stats the signal file and only a file found there is consumed (removed) and planned through `onSignal`; an event that finds no file runs nothing.
A cycle runs `list-sessions` as today, then calls `afterCycle` with whether the set equals the previous cycle's; idle exit and orphan reap keep their counts of three consecutive cycles.
When `open` fails the loop logs that once at Warn and keeps its backed-off cadence with no event wake.
`watchdogCmd` passes `reedengine.OpenFileWatch`.
`watchdog_test.go` gains a table test over `discoveryWake`: the cadence doubles to the ceiling across unchanged cycles, resets on a changed set and on a signal, a signal within the base of the previous cycle runs its cycle when the base has passed, and a signal after it runs at once; `TestWatchdogDefaultTiming` takes the ceiling and asserts the periodic fields are at or above one second, the D2 floor assertion for this package.
The two integration tests pass an opener and fill `DiscoveryCeiling` in their compressed timings; `TestWatchdogDaemon` adds a step asserting that a session booted through reed is discovered within one base cadence after its boot, through the signal the boot touches.

**ImpactSummary:** The daemon's outer loop changes its wait mechanism and gains an opener parameter whose callers are the verb and two integration tests; the loop's observable cycle work is unchanged.
