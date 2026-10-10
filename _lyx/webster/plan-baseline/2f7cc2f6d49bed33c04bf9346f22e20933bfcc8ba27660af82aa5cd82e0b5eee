# Card 6 — watchdog-discovery-backoff

**Create:**
- `internal/reedcli#watchdogHubDiscoveryCeiling`
- `internal/reedcli#discoveryWake`
- `internal/reedcli#discoveryWake.afterCycle`
- `internal/reedcli#discoveryWake.onSignal`
- `internal/reedcli#discoveryWake.next`
- `internal/reedcli#sameSessionSet`
- `internal/reedcli#consumeDiscoverSignal`
- `internal/reedcli/doc.go`

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
- `internal/reedengine#NextWakeCadence`
- `internal/reedengine#DiscoverSignalFileName`
- `internal/reedengine#OpenFileWatch`

**Intent:** The discovery loop backs off while the session set is unchanged and wakes on the discover signal (D5, the `reedcli` half).
`watchdogTiming` gains `DiscoveryCeiling`, filled from `watchdogHubDiscoveryCeiling` by `watchdogDefaultTiming`; `watchdogHubDiscoveryCycle` stays the 5 s base and its comment names the backoff.
`discoveryWake` is the pure wake planner: it holds the last cycle's time and the current cadence; `afterCycle` records a cycle and moves the cadence with `reedengine.NextWakeCadence` (reset to the base when the set changed, doubling to the ceiling otherwise); `onSignal` marks a consumed signal pending; `next` returns how long to wait from `now`, the time until the pending signal's cycle, which is never sooner than the base after the last cycle, else until the cadence ends.
`sameSessionSet` compares two listings as sets.
`runWatchdogLoop` replaces its ticker with a timer re-armed from `next` after every wake, opens the hub-level discover signal through `open` on `fabricengine.HubScratchDir(hub)` for `reedengine.DiscoverSignalFileName`, and selects over the timer, the events and `ctx.Done()`.
An event wake calls `consumeDiscoverSignal`, which stats the signal file and removes it when found, answering whether it consumed one; only a consumed signal is planned through `onSignal`, and an event that finds no file runs nothing.
Bound: the discover signal is a new routing edge into the daemon, and any process of the operator can touch the hub scratch dir; a signal wake never runs a cycle sooner than the 5 s base after the previous one, so a toucher in a loop costs one `list-sessions` per 5 s per hub, today's cadence, and lyx's own writer touches once per booted session (D5's Bound).
A cycle runs `list-sessions` as today, then calls `afterCycle` with whether the set equals the previous cycle's; idle exit and orphan reap keep their counts of three consecutive cycles.
When `open` fails the loop logs that once at Warn and keeps its backed-off cadence with no event wake.
`watchdogCmd` passes `reedengine.OpenFileWatch`.
`watchdog_test.go` gains a table test over `discoveryWake`: the cadence doubles to the ceiling across unchanged cycles, resets on a changed set and on a signal, a signal within the base of the previous cycle runs its cycle when the base has passed, and a signal after it runs at once; a test over `consumeDiscoverSignal` in `t.TempDir()` asserts a present file is consumed and removed and an absent one answers false and leaves nothing, the tier-1 proof of the event branch, since `runWatchdogLoop` calls `reedengine.ListSessions` against a live tmux server and so runs only in the `tmux` tier; `TestWatchdogDefaultTiming` takes the ceiling and asserts the periodic fields are at or above one second, the D2 floor assertion for this package.
`internal/reedcli` has no package doc, so this card creates `doc.go` for it: the package comment says it is the reed CLI, whose domain lives in `internal/reedengine`'s package doc, and states the watchdog daemon's discovery loop: the cadence from 5 s doubling to 60 s across unchanged cycles and reset by a changed set, the discover-signal wake and its base-cadence bound, and that idle exit and orphan reap now take up to three minutes at the backed-off cadence.
`cmd/lyx/testdata/test-budget.yaml` raises `internal/reedcli` by the two new top-level tests.
The two integration tests pass `reedengine.OpenFileWatch` and fill `DiscoveryCeiling` in their compressed timings; `TestWatchdogDaemon` adds a step asserting that a session booted through reed is discovered within one base cadence after its boot, through the signal the boot touches.

**ImpactSummary:** The daemon's outer loop changes its wait mechanism and gains an opener parameter whose callers are the verb and two integration tests; the loop's observable cycle work is unchanged.
