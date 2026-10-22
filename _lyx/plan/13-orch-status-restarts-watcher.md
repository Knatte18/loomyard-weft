# Card 13 — `lyx orch status` restarts a dead watcher and names a pane elsewhere

**Edit:**
- `internal/orchcli#statusFields`
- `internal/orchcli#orchCLI`
- `internal/orchcli#orchCLI.statusCmd`
- `internal/orchcli#orchCLI.stopCmd`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli#fakeStrands`
- `internal/orchcli#fakeStrands.RemoveStrand`
- `internal/orchcli#fakeStrands.Strands`
- `internal/orchcli#newTestCLI`
- `internal/orchcli#TestStatus_Envelope`
- `internal/orchcli#TestStop`
- `internal/orchcli#TestStart_DeadStrandRemovedBeforeStart`
- `internal/orchcli/doc.go`

**Create:**
- `internal/orchcli#orchCLI.restartWatcher`
- `internal/orchcli#TestStatus_RestartsADeadWatcher`

**Uses:**
- `internal/orchengine#Paths`
- `internal/orchcli#orchCLI.spawnWatcherProcess`
- `internal/orchcli#newStartHarness`
- `internal/orchcli#startHarness`
- `internal/reedengine#StrandStatus`

**Intent:**
D3: `status` is what the orch and the operator run when the orch goes quiet, so it restarts a dead watcher; D2: it shows where a moved-out orch pane went.

`statusCmd`, once it has read the state, the strands and the watch lock, calls `restartWatcher` when the orch strand is tracked and live per D1 (`strand.Live`, which a shell-only orch past the grace is not) and no watcher holds `watch.lock`.
`restartWatcher` takes `start.lock` with `lock.TryAcquireWriteLock` and, when another process holds it, restarts nothing and answers false, so `status` never races a `start` or a `stop`.
Holding it, it reads the state, the strands and the watch lock again, since a `stop` or `start` may have run between `status`'s first read and the lock, and spawns only when the recorded strand is still tracked and live and no watcher holds `watch.lock`; otherwise it answers false.
It then spawns the watcher through `c.spawnWatcher`, logs `orch: restarted a dead watcher` at Info with the previous watcher's recorded exit (`State.WatcherExit`), releases the lock and answers true.
A spawn failure is returned and `status` prints the error envelope with the way forward `lyx orch start`, which spawns a watcher for a live strand.
The envelope gains `watcher_restarted`, true only when this call spawned one; `watcher_live` keeps reporting what the probe found.
When a live strand has no watcher and the restart was skipped because `start.lock` was held, the envelope adds `hint`: a `start` or `stop` is running, so rerun `lyx orch status` once it ends, or run `lyx orch start`.
A dead strand gets no watcher from `status`: `start` relaunches strand and watcher together.
Bound: `status` writes only this one spawn; two `status` calls in sequence, the second before the first one's watcher takes `watch.lock`, may spawn two watchers, and `watch.lock` keeps one (the second exits `already_running`).

`stopCmd` takes `start.lock` (creating `c.paths.Dir` first, as `start` does) around its strand read and removal and releases it after, so a `status` restart, which re-reads under the same lock, never binds a watcher to a strand a concurrent `stop` removes.
It waits for the lock with `lock.AcquireWriteLockWithin`, bounded by a named constant of one minute, since a `start` holds it across its origin reconcile and session launch; past the bound it refuses with the error envelope naming the held `start.lock` and the way forward, rerun `lyx orch stop` once the running `start` ends, so `stop`, the way forward `start`'s own refusals name, never hangs.
The refusal is reached by a test holding `start.lock` with the bound shortened through an `orchCLI` field the test sets, as the start harness sets its sleep.
`startCmd`'s `StartSpawnWatcher` branch keeps spawning a watcher for a live strand and logs the same `orch: restarted a dead watcher` line with `st.WatcherExit` before the spawn.
A shell-only orch strand now reads dead, so `start` takes `StartRelaunch` and `start --continue` resumes the recorded session in one step, with no code change here (D1's acceptance).

`statusFields` gains `strand_elsewhere`: the tracked strand's `StrandStatus.Elsewhere` (card 5), empty when untracked or when none, so it shows until `start`'s relaunch removes the strand.
The `status` and `stop` `Long` help name `watcher_restarted`, `strand_elsewhere` and the lock; `internal/orchcli/doc.go` states the restart, its bound and stop's lock.

Tests:
- `newTestCLI` sets `Paths.StartLockPath` under the test's orch directory and a counting `spawnWatcher`; `fakeStrands` gains `onRemove func()`, which `RemoveStrand` runs before it records the removal.
- `TestStatus_RestartsADeadWatcher` is new and not parallel, since `logcapture.CaptureVerbose` swaps the process-global logger to read the Info line: a live tracked orch strand with no watcher lock held spawns once and answers `watcher_restarted: true`, and the log carries the line with the recorded exit; with `start.lock` held by the test it spawns nothing and answers `watcher_restarted: false` with the hint; with a watcher holding `watch.lock` it spawns nothing; with the strand tracked but not live it spawns nothing; with no recorded strand it spawns nothing; with the fake's strand list emptied by the `Strands` call after the first (the strand a concurrent `stop` removed before the lock) it spawns nothing; with a failing spawn it answers the error envelope naming `lyx orch start`.
  The last row needs `fakeStrands` to answer a second list, so it gains `after []reedengine.StrandStatus`, which `fakeStrands.Strands` answers from its second call on when set.
  The same test drives `start`'s `StartSpawnWatcher` branch through `newStartHarness` with a live strand, no watcher and a recorded `WatcherExit`, and asserts the restart line with that exit, since `TestStart_LiveStrand` is parallel and cannot capture the log.
- `TestStatus_Envelope` gains `strand_elsewhere: ""` in its expected maps, `watcher_restarted: true` in the rows whose orch strand is live with no watch lock held (populated state, too short pane) and `watcher_restarted: false` in the row with no strand, and a row with a tracked strand carrying `Elsewhere: "other:2"` answering `strand_elsewhere: "other:2"`.
- `TestStop` sets `onRemove` to try `lock.TryAcquireWriteLock(StartLockPath)` and asserts the try fails, so the removal runs under the lock, and asserts the lock is free after `stop` returns; a row holding `start.lock` with the wait bound shortened answers the error envelope naming `start.lock` and removes nothing.
- `TestStart_DeadStrandRemovedBeforeStart` gains a `--continue` row for D1's acceptance: a tracked orch strand reading `Live: false, Present: true, OwnWindow: true` (a shell-only orch past the grace) with a recorded session id is removed once and relaunched once in that run, with the spec's `ResumeSessionID` the recorded id.

**ImpactSummary:** `status` gains one guarded spawn and two envelope keys, `stop` takes the lock `start` already takes, and `start` gains one log line; the watcher's own lock rules are unchanged and nothing outside orchcli reads these envelopes.

**Commit:** 13: lyx orch status restarts a dead watcher and names a pane elsewhere
