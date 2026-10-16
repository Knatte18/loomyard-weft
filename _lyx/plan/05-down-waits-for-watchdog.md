# Card 5 — The down verb waits, bounded, for the watchdog to leave its loop

**Create:**
- `internal/reedcli#watchdogDownWait`
- `internal/reedcli#watchdogDownPoll`
- `internal/reedcli#probeWatchdogLock`
- `internal/reedcli#waitWatchdogLockFree`
- `internal/reedcli#fakeStrandOps.Down`
- `internal/reedcli#TestDownCmd_WaitsForTheWatchdogOnlyWhenItEndedTheServer`

**Edit:**
- `internal/reedcli#strandOps`
- `internal/reedcli#reedCLI`
- `internal/reedcli#reedCLI.downCmd`
- `internal/reedcli#fakeStrandOps`
- `internal/reedcli#TestWatchdogDaemon`
- `internal/reedcli/doc.go`

**Uses:**
- `internal/reedcli#watchdogLockFileName`
- `internal/reedcli#reedCLI.strandEngine`
- `internal/reedcli#RunCLIIn`
- `internal/lock#TryAcquireWriteLock`
- `internal/testkit/logcapture#Capture`

**Intent:**
D3, in the verb.
`strandOps` gains `Down() (reedengine.DownResult, error)`, which `*reedengine.Engine` already has, and `downCmd` calls `c.strandEngine().Down()` instead of `c.eng.Down()`, so a test drives the verb over `fakeStrandOps`.
`fakeStrandOps` gains a scripted `down reedengine.DownResult` and `downErr error`, returned by its new `Down` method.

After `Down` succeeds, `downCmd` calls `waitWatchdogLockFree` whenever the result's `EndedServer` is true.
Every reed CLI engine is built from `hubgeom.ReedGeometry` with `c.hubPath` set, so the verb has no standalone mode; the standalone half of the acceptance criteria is carried by card 3's no-path-told row.
`reedCLI` gains the test seams `downLockWait`, `downLockPoll`, `downLockProbe` and `downLockSleep`; zero or nil means `watchdogDownWait`, `watchdogDownPoll`, `probeWatchdogLock` and `time.Sleep`, so the bound and the poll interval are injectable and a test never waits them out.
A `Down` error writes the error envelope as today and waits for nothing; a `down` that left a sibling session (`EndedServer` false) waits for nothing, so the watchdog keeps serving the sibling.
The envelope is written after the wait and is unchanged in every case.

`probeWatchdogLock` calls `lock.TryAcquireWriteLock` on the path and, when it acquires, releases the lock at once and answers free; contention answers not free; an error is returned.
`waitWatchdogLockFree` probes `filepath.Join(fabricengine.HubScratchDir(hub), watchdogLockFileName)`, sleeping `poll` between probes, until a probe answers free or the probes have spanned `wait`, which is `wait/poll + 1` probes:
a free probe returns;
a probe error (no scratch directory, so no daemon can hold the lock) logs at Debug and returns;
when the probes run out it logs one Warn naming the hub and the lock path and returns.
It logs nothing per probe (`PATTERN-log-levels`).
`watchdogDownWait` and `watchdogDownPoll` are declared in `internal/reedcli/watchdog.go` beside `watchdogLockFileName`, and the line declaring `watchdogDownPoll` carries `//lyx:one-shot bounded lock poll of at most watchdogDownWait`, the form `internal/orchengine/watchloop.go` uses for `watchLockRetryInterval`: `watchdog.go` is listed in `cmd/lyx/daemonwakeups_test.go`'s `unboundedWaitFiles`, and that scan fails a bare `100 * time.Millisecond` there.

Bound: the wait is at most `watchdogDownWait` (`PATTERN-daemon-wakeups`) and never fails `down`, which stays idempotent; the probe holds the lock only between its acquire and its release, so a daemon spawned in that instant loses the race and exits, and the hub's next `up`, `attach` or `resume` spawns one.
A free lock means the daemon has left its loop; the exiting process may still be running card 4's one re-check listing and at most one successor spawn when `down` returns.
A daemon finishing an in-flight orphan reap holds the lock until the reap ends, so such a `down` can reach the bound and warn, and still succeeds.

`TestDownCmd_WaitsForTheWatchdogOnlyWhenItEndedTheServer` drives `downCmd` over `fakeStrandOps` with a temp hub, `downLockWait` 1s and `downLockPoll` 100ms, `downLockSleep` recording its calls without sleeping, and logs captured with `logcapture.Capture`.
Each row asserts the envelope (the ok `{"session": ...}` payload, or the error envelope), the number of probes, and whether one Warn naming the hub and the lock path was logged:

- ended the server, the real `probeWatchdogLock` over a free lock: ok envelope, one probe, no Warn, and a fresh `lock.TryAcquireWriteLock` on the lock path acquires afterwards, so the probe released what it took.
- ended the server, the real probe with the hub scratch directory absent: ok envelope, one probe, no Warn.
- ended the server, a scripted probe answering held, held, free: ok envelope, three probes, two sleeps of 100ms, no Warn.
- ended the server, a scripted probe always held: ok envelope unchanged, eleven probes, exactly one Warn naming the hub and the lock path.
- left a sibling (`EndedServer` false): ok envelope, no probe, no Warn.
- `Down` error: error envelope, no probe, no Warn.

`TestWatchdogDaemon`'s `DownStopsTheDaemonAndReleasesLock` step (card 4) runs its last-session `down` as `lyx reed down` through `RunCLIIn` in the prime worktree instead of the engine's `Down`, and asserts the verb's ok envelope and that a `lock.TryAcquireWriteLock` on the lock path acquires as soon as the verb returns, with no further wait: the verb end to end, the stop and the bounded wait proven together against a live daemon.

Docs: `downCmd`'s Long text says that when `down` ends the hub's server it asks the watchdog to stop and waits up to ten seconds for it to leave, warning and still succeeding on timeout; `internal/reedcli/doc.go` states the same beside the daemon paragraph.

**ImpactSummary:** `downCmd` gains a bounded lock poll on its last-session path and routes `Down` through the existing `strandOps` seam; `strandOps` has one production implementer (`*reedengine.Engine`) and one fake, both covered here, and one live test step moves onto the verb.

**Commit:** 5: reed down waits, bounded, for the watchdog to leave its loop
