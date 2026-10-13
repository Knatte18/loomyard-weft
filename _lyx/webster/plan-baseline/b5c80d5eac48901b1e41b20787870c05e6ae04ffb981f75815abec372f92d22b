# Card 5 — The watchdog retires on a replaced binary; lock contention exits at Debug

**Create:**
- `plan:internal/reedcli#errWatchdogBinaryReplaced` -> `var errWatchdogBinaryReplaced = errors.New("reed: the watchdog's binary was replaced")`
- `plan:internal/reedcli#exeReplaced` -> `func exeReplaced(start, now os.FileInfo, statErr error) bool`
- `plan:internal/reedcli#exeWatch` -> `func exeWatch(stat func() (os.FileInfo, error)) func() bool`
- `plan:internal/reedcli#TestExeReplaced` -> `func TestExeReplaced(t *testing.T)`
- `plan:internal/reedcli#TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping` -> `func TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping(t *testing.T)`

**Edit:**
- `internal/reedcli#runWatchdogLoop` -> `func runWatchdogLoop(ctx context.Context, hub, tmuxPath, shellPath string, timing watchdogTiming, open reedengine.FileWatchOpener, replaced func() bool) error`
- `internal/reedcli#reedCLI`
- `internal/reedcli#reedCLI.watchdogCmd`
- `internal/reedcli#TestWatchdogDaemon`
- `internal/reedcli#startReapLoop`
- `internal/reedcli/doc.go`
- `internal/reedengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/reedengine#SpawnWatchdog`

**Intent:**
D1, D2 and D4: the hub watchdog daemon retires and respawns itself when the `lyx` binary it was started from is replaced, so a deploy never leaves an old daemon laying out sessions with stale render code; and losing the lock race logs at Debug.

`exeReplaced` is the pure comparison: a nil `start` (no baseline), or a non-nil `statErr`, answers false; a `now` that is not `os.SameFile` as `start`, or whose modification time differs from `start`'s, answers true; anything else answers false.
`exeWatch` stats once at startup through `stat` to record the baseline, logging a failed baseline stat at Debug and leaving the baseline nil, and returns the check the loop calls: it stats again, logs a failed stat at Debug, and answers `exeReplaced(baseline, now, err)`.
A failed stat therefore counts as unchanged, so a binary missing mid-build never retires the daemon.

`runWatchdogLoop` gains `replaced`, called once on each timer tick, the existing discovery tick, before `ListSessions`: when it answers true the loop returns `errWatchdogBinaryReplaced`, and its existing deferred teardown cancels every watched session and waits for in-flight reaps.
A nil `replaced` is never called, so the test callers `TestWatchdogDaemon` and `startReapLoop` pass nil and keep their behavior.
The check adds no wakeup of its own (`PATTERN-daemon-wakeups`) and runs only on the timer branch, never on a discover-signal wake.

`reedCLI` gains two seams, nil in production: `watchdogExeStat func() (os.FileInfo, error)`, defaulting to an `os.Stat` of `os.Executable()`'s path, and `respawnWatchdog func(hubPath, tmuxPath, shellPath string, suppress bool)`, defaulting to `reedengine.SpawnWatchdog`.
RunE passes `testing.Testing()` as `suppress`, the not-under-`go test` guard computed at that call site as every other caller's is, so the seam sees the value production would hand `SpawnWatchdog`.
`watchdogCmd`'s RunE builds the check with `exeWatch` before the loop starts and passes it in.
When the loop returns `errWatchdogBinaryReplaced`, RunE releases the hub watchdog lock first (`fl.Release`; the deferred release that follows is a no-op on an unlocked flock), then calls the respawn seam with its own `--hub-path`, `--tmux` and `--shell` values, then logs `reed: watchdog retired; its binary was replaced and a successor was spawned` at Info and returns nil, so the process exits 0.
The respawn is the existing best-effort detached spawn, which logs its spawn as `PATTERN-spawn-observability` requires; a failed respawn leaves the hub without a daemon until the next `reed up`, `attach` or `resume`.
Bound: each daemon process retires at most once, since it exits; a deploy that rewrites the file in several steps can chain retirements, one per observed change, ending when the file stops changing.
The check only retires the process and never touches config, so `PATTERN-binary-change-reconcile` is unaffected.
The lock-contention line `reed: watchdog lock already held, exiting` moves from Info to Debug, so at the default level the lazily opened durable sink writes nothing and the short-lived process creates no trace file.
The verb's `Long` help, `reedcli/doc.go` and `reedengine/doc.go`'s watchdog-daemon account say the daemon retires and respawns on a replaced binary, and that a daemon started before this change carries no such check and is ended once by hand after the deploy (D18).

`TestExeReplaced` is table-driven over real temp files: the same unchanged file answers false; a stat error answers false; a nil baseline answers false; a different file answers true; the same file with its modification time moved by `os.Chtimes` answers true.

`TestWatchdogCmd_ExitsZeroBeforeOrInsteadOfLooping`, in `watchdog_test.go` (untagged: neither row reaches tmux), drives `watchdogCmd` against a temp hub whose `fabricengine.HubScratchDir` it creates, serially, since the command points the process-global durable sink at the hub's logs directory (each row's cleanup calls `logger.SetDurableSinkDir("")` and `logger.SetVerbosity(0)`):
- "lock already held": the test holds the hub watchdog lock through `lock.TryAcquireWriteLock` first, runs the command at the default verbosity, and asserts it returns nil, the respawn seam was not called, and `fabricengine.HubLogsDir(hub)` holds no file.
- "binary replaced": `watchdogExeStat` answers the stat of one temp file on its first call and of a second temp file after, `watchdogTiming` is a one-millisecond cycle, `--tmux` and `--shell` are fixed strings; the command returns nil; the respawn seam was called exactly once, with the hub, tmux and shell values and `suppress` true, so under `go test` the retirement spawns nothing, since `SpawnWatchdog` returns at once on a set suppress (`TestSpawnWatchdog_ReturnsWithoutSpawning` in `internal/reedengine`);
the seam fake, when called, takes the hub watchdog lock with `lock.TryAcquireWriteLock` and records that it was acquired (releasing it again), which pins that RunE released the lock before the respawn and not only through its deferred release;
and the hub logs directory's trace file holds the `reed: watchdog retired` line.
`cmd/lyx/testdata/test-budget.yaml`'s `internal/reedcli` row rises from 38 to 40.

**ImpactSummary:** one re-signed unexported loop function with three callers, all in `internal/reedcli` and listed here, two nil-defaulted CLI seams and one log level; the daemon's discovery, reap and idle-exit behavior is unchanged when the binary is not replaced.

**Commit:** 5: reed: retire and respawn the watchdog on a replaced binary
