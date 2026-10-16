# Card 4 — The watchdog honours a stop, re-checks on exit and respawns only for a live hub

**Create:**
- `internal/reedcli#watchdogStopFloor`

**Edit:**
- `internal/reedcli#watchdogHubIdleCycles`
- `internal/reedcli#consumeDiscoverSignal` -> `func consumeDiscoverSignal(path string) (consumed, stop bool, err error)`
- `internal/reedcli#discoveryWake`
- `internal/reedcli#discoveryWake.onSignal` -> `func (d *discoveryWake) onSignal(now time.Time, stop bool)`
- `internal/reedcli#discoveryWake.afterCycle`
- `internal/reedcli#discoveryWake.next`
- `internal/reedcli#watchdogTiming`
- `internal/reedcli#watchdogDefaultTiming`
- `internal/reedcli#runWatchdogLoop` -> `func runWatchdogLoop(ctx context.Context, hub, tmuxPath, shellPath string, timing watchdogTiming, open reedengine.FileWatchOpener, list func(tmuxPath, socketKey string) ([]string, error), replaced func() bool) error`
- `internal/reedcli#reedCLI`
- `internal/reedcli#reedCLI.watchdogCmd`
- `internal/reedcli#TestConsumeDiscoverSignal`
- `internal/reedcli#TestDiscoveryWake`
- `internal/reedcli#TestWatchdogDefaultTiming`
- `internal/reedcli#TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping`
- `internal/reedcli#runWatchdogCmdInBackground` -> `func runWatchdogCmdInBackground(t *testing.T, hub, tmuxPath string, timing watchdogTiming) (cancel context.CancelFunc, done chan error, out *bytes.Buffer)`
- `internal/reedcli#TestWatchdogDaemon`
- `internal/reedcli#startReapLoop`
- `internal/reedcli/doc.go`

**Uses:**
- `internal/reedengine#DiscoverStopMarker`
- `internal/reedcli#watchdogLockFileName`
- `internal/reedcli#errWatchdogBinaryReplaced`
- `internal/reedcli#compressedWatchdogTiming`
- `internal/fsx#AtomicWriteBytes`
- `internal/lock#TryAcquireWriteLock`

**Intent:**
D2, D3a and D4, in the daemon.

`consumeDiscoverSignal` reads the file before removing it and reports `stop` true when the content, whitespace-trimmed, equals `reedengine.DiscoverStopMarker`.
An absent file answers `false, false, nil`; an empty file (a cold boot's touch, which truncates a pending stop) answers `true, false, nil`; a read error other than not-exist, or a removal error after a successful read, is returned with `consumed` and `stop` false.
A stale marker a daemon finds later is harmless: it ends nothing unless the stop-woken cycle's listing is idle.

`watchdogTiming` gains `StopFloor`, and `watchdogDefaultTiming` fills it from `watchdogStopFloor`, one second, the `PATTERN-daemon-wakeups` floor.
`discoveryWake` gains `stopFloor` and a pending `stop` flag; `onSignal(now, stop)` records the signal as today and ORs `stop` into the flag, so a discover landing after a stop and before the cycle keeps the stop; `afterCycle` clears the flag with the signal; and `next` puts a pending signal's cycle at the later of the signal and `lastCycle + stopFloor` when the flag is set, else `lastCycle + base` as today.

`runWatchdogLoop` takes `list`, the session lister (production passes `reedengine.ListSessions`), and one cycle runs in this order:
the cycle notes whether a stop was pending, lists the hub's sessions through `list(tmuxPath, reedengine.ServerName(hub))`, and moves the wake as today.
Then the replaced check runs, moved after the listing (D4): replaced with an affirmative listing returns `errWatchdogBinaryReplaced`; replaced with an idle listing logs at Info that the daemon retires on a replaced binary with no successor of its own and returns nil.
Then an idle listing on a stop-woken cycle logs at Info that the daemon exits on a stop request and returns nil at once, without waiting for `IdleCycles`; an idle listing otherwise counts toward `IdleCycles` as today; an affirmative listing proceeds as an ordinary cycle and the daemon keeps running.
A cancelled context still returns `ctx.Err()`.
A daemon whose watch failed to open never sees a stop and exits by the idle-cycle rule as today.
`runWatchdogLoop`'s doc comment is rewritten to match: its return conditions name the stop exit and the idle replaced retire, and the replaced check is described as running after the session listing on each timer tick.
`watchdogHubIdleCycles`' comment says the three cycles cover a `down` followed by an `up` for a daemon whose watch failed to open and so never sees the stop, and a transient list-sessions error, which `sessionsAreIdle` counts as idle.

`reedCLI` gains the test seam `watchdogListSessions func(tmuxPath, socketKey string) ([]string, error)`, nil meaning `reedengine.ListSessions`.
`watchdogCmd`'s RunE passes it to the loop and, after the loop returns:

- `errWatchdogBinaryReplaced`: releases the lock and respawns, as today.
- nil (a stop, the idle-cycle rule or an idle replaced retire): releases the lock, then, unless `cmd.Context()` is done, lists the hub's sessions once more through the same lister; an affirmative listing (`!sessionsAreIdle`) calls the respawn seam (`c.respawnWatchdog`, else `reedengine.SpawnWatchdog`, with `testing.Testing()` as today) and logs at Info that a successor was spawned for a live hub; an idle listing spawns nothing (D3a).
- any other error (a cancelled context): logs as today and spawns nothing, whatever a listing would say.

The exit-path table's stop-with-an-idle-listing-and-a-cancelled-context row is the test of the nil branch's `cmd.Context()` guard.

Bound: the stop lets a writer skip the idle-cycle tolerance and shorten the cadence floor to `StopFloor`, never below one second in production; the exit re-check costs one list-sessions and at most one spawn per daemon exit, and the successor is single-instance by the lock.
`sessionsAreIdle` counts a list error as idle, so a transient error on a stop-woken cycle can end a daemon whose hub still has a session; the re-check bounds that to one successor spawn.

Tests:
`TestConsumeDiscoverSignal` adds rows: a file holding `DiscoverStopMarker` answers `true, true` and is removed; an empty file answers `true, false`.
Its non-empty-directory row now fails the read before any removal and is relabelled the read-error row; a new row reaches the removal error with a readable file inside a directory made read-only, skipped on Windows and when running as root, where the removal would succeed.
Every row asserts the new `stop` result.
`TestDiscoveryWake` builds its wake with a one-second `stopFloor`, calls `onSignal(now, false)` in its existing rows, and adds rows: a stop 300ms after the last cycle waits until one second after it; a stop two seconds after it runs at once; a stop then an ordinary signal before the cycle still waits only for the stop floor; an ordinary signal then a stop does the same; a cycle after a stop restores the base floor for the next ordinary signal.
`TestWatchdogDefaultTiming` adds `StopFloor` to its at-least-one-second map and its expected value.

`TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping` becomes the table of `watchdogCmd`'s exit paths; its doc comment says so.
Its table gains, per row, `IdleCycles`, a lister script answering by call number (idle as an empty listing, affirmative as one session name), and whether the test writes the stop.
Rows keep `DiscoveryCycle` and `DiscoveryCeiling` at 1ms and add `StopFloor` 1ms and `OrphanGoneCycles` 1000 so no affirmative cycle reaps; the real `reedengine.OpenFileWatch` watches the hub scratch directory; each row is bounded by a 10-second wait on the command's return.
The test writes the stop the way `Engine.writeDiscoverStop` does, `DiscoverStopMarker` through `fsx.AtomicWriteBytes` onto the signal path, once the lister has been called so the watch is open.
The empty-logs-directory assertion is limited to the lock-already-held row, the only row that runs no loop.
Every other row asserts its own trace lines instead: for example the stop-with-an-idle-listing row the exit-on-stop Info line, and the rows with no successor that no successor-spawned line appears.
Rows:

- lock already held: unchanged.
- binary replaced, every listing affirmative: one respawn with the lock free and the "watchdog retired" trace, as today.
- idle-cycle exit (`IdleCycles` 3, every listing idle): no respawn.
- idle-cycle exit, re-check affirmative (the call after the third idle cycle answers a session): one respawn, lock free.
- stop with an idle listing (`IdleCycles` 1000000, so the idle-cycle rule cannot end the run within the bound): the command returns, the re-check is idle, no respawn.
- stop with an affirmative listing (`IdleCycles` 1000000, every call affirmative): after the marker is consumed (the file is gone) and two more lister calls, the second proving the stop-woken cycle finished and the loop started another, the command is still running; the test cancels its context and the command returns with no respawn, though the lister still answers a session.
- stop with an idle listing and a cancelled context (`IdleCycles` 1000000): the lister script cancels the command's context during the stop-woken cycle's call, answers idle on that call and affirmative on every later call; the loop returns nil, and the row asserts no respawn and no lister call after the cancelling one.
- binary replaced, idle cycle listing and idle re-check (`IdleCycles` 1000000): no respawn, the "retires on a replaced binary with no successor" Info line is in the trace and the "watchdog retired" line is not.
- binary replaced, idle cycle listing and affirmative re-check (`IdleCycles` 1000000): one respawn, lock free.

The affirmative rows' session name names no directory under the hub, so each affirmative cycle's `enterSession` fails at `lyxcwd.ResolveWorktree`; that spawns git under the package's `TestMain`, which arms `gitkit.HermeticGitEnv()`, and only the stop-with-affirmative row runs such cycles, a handful before its cancel.

`TestWatchdogDaemon` and `startReapLoop` pass `reedengine.ListSessions` to `runWatchdogLoop`.
`runWatchdogCmdInBackground` takes the timing its daemon runs; `TestWatchdogDaemon` passes `compressedWatchdogTiming()` in its lock-and-logs step.
Its `ExitsAfterIdleCyclesAndReleasesLock` step becomes `DownStopsTheDaemonAndReleasesLock`: it boots the prime and the pair sessions and starts the daemon with compressed timing and `IdleCycles` 1000, so only a stop can end it within the step.
It runs the pair engine's `Down`, a non-last `down`, and asserts the daemon is still running after five discovery cycles; it then runs the prime engine's `Down`, the last session, and asserts the daemon command returns within the step's 10-second bound and a `lock.TryAcquireWriteLock` on the lock path then acquires.
So a non-last `down` is proven against a live daemon that can see a stop, and the last one's stop ends it.
The shared discovery loop of the later steps opens its watch through a `FileWatchOpener` that returns an error, so those steps run a daemon that sees no stop and keeps today's idle-cycle rule: `DownThenUpDoesNotKillDaemon` still proves the idle cycles cover a down followed by an up.
The file header's list of live claims replaces "the idle-exit timer" with the stop a last `down` writes.

Docs: `internal/reedcli/doc.go` states the stop request (a stop-woken idle cycle exits at once, never sooner than one second after the previous cycle), the exit re-check that spawns a successor only for a live hub, and the replaced binary retiring with a successor only when its cycle's listing found a session.
`watchdogCmd`'s Long text says the daemon retires on a replaced binary and respawns only while the hub has a session, and that `down` asks it to stop.

**ImpactSummary:** Confined to `internal/reedcli`'s daemon: two re-signed private functions and one re-signed method with every caller (`watchdogCmd`, `TestWatchdogDaemon`, `startReapLoop`, three unit tests) updated in this card; no exported surface changes.

**Commit:** 4: Watchdog exits on a stop request, re-checks on exit and respawns only for a live hub
