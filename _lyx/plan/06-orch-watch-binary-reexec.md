# Card 6 — orch-watch-binary-reexec

**Edit:**
- `internal/orchengine#Watcher`
- `internal/orchengine#Watcher.Run`
- `internal/orchengine/doc.go`
- `internal/orchcli#orchCLI.watchCmd`
- `internal/selfexec/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/orchengine#Watcher.WatchBinary` -> `func (w *Watcher) WatchBinary(check func())`
- `plan:internal/orchengine#Watcher.restartSafe` -> `func (w *Watcher) restartSafe() bool`
- `plan:internal/orchengine#TestRun_ChecksTheBinaryOnlyAtARestartSafeTick` -> `func TestRun_ChecksTheBinaryOnlyAtARestartSafeTick(t *testing.T)`

**Intent:**
`lyx orch watch` re-execs itself onto a replaced binary, at a point where nothing in flight is lost (D8).

`Watcher` gains a `binaryCheck func()` field, nil by default, and `WatchBinary(check)` sets it, as `WatchPause` sets the pause reading.
`Watcher.Run` calls it after a tick that succeeded and returned not done, before the poll sleep, only when `restartSafe` reports true.
So the check adds no wake of its own (`PATTERN-daemon-wakeups`), and runs at most once per poll.
`selfexec.Watcher.Check` already logs its repeated stats at Debug (`PATTERN-log-levels`).

`restartSafe` re-reads the state from disk after the tick.
It is true only when the phase is idle, so no cycle step is in flight and the persisted cycle state carries over.
It also needs the pause record at `w.pause.RecordPath` to read with no live hub entry at all, handled or not, so no in-memory pause handling is live to be lost: the handled set and the first-Esc marks are empty of live entries.
A watcher never told a pause reading reads no entry.
A state or record read error is not safe, so the check waits for the next tick.
An entry the old image settled during its grace is never handed to the new image, because the check never fires while that entry is live.

`orchCLI.watchCmd` wires `watcher.WatchBinary(selfexec.New().Check)` beside `WatchPause`.
The exec replaces the process with the same argv and environment, and the watch lock does not survive it, since Go opens the lock file close-on-exec.
The new image takes the lock again through `acquireWatchLock`'s existing retry.
Bound: in that gap `lyx orch start` may spawn a second watcher; both contend for the one lock and the loser exits as already running, so exactly one watcher remains.
That contention is pinned by `internal/orchengine#TestRun_ErrWatcherRunningWhileLockHeld`, and the retry by `TestRun_TakesLockReleasedBetweenRetries`.
A `refresh`, `distill`, hub `pause` or `revive` in the gap reports no live watcher, as it does with none.
The new image holds the cycle machine under a live hub entry from its first tick, and interrupts a busy orch only for an entry whose grace end is still ahead.

`doc.go` of `orchengine` states the re-exec point and the gap;
`selfexec`'s doc no longer says its callers are pane processes only, and names the orch watcher daemon.

`TestRun_ChecksTheBinaryOnlyAtARestartSafeTick` in `watchloop_test.go` drives `Run` with a `check` that counts its calls, standing in for `selfexec`'s exec seam, since a test binary never execs.
The `sleep` closure passed to `Run` counts the polls and sets the `fakeSession`'s `alive` field false after the row's third one, so the next tick reports the strand gone and `Run` returns.
The check runs before that sleep, so the count it sees is per tick.
Rows:
- idle phase, no pause record: the check runs once per tick that returned not done;
- a non-idle phase, `PhaseHandoffRequested`, saved before `Run`: the check never runs;
- a live hub entry in the told record, handled by the watcher or not yet: the check never runs;
- a live slug entry only: the check runs, since no hub entry is live.
- the state unreadable after a successful tick: on the second tick, the `fakeSession`'s `onAlive` hook, which `StrandAlive` runs before the tick reads events, moves the state file aside and puts a directory at the state path.
  An idle tick with no events, no notices and no cycle trigger saves nothing after that point, so the tick succeeds and `restartSafe`'s re-read fails; the `sleep` closure puts the state file back.
  The check runs on the first and third ticks only.
- the told pause record unreadable: a directory at `RecordPath`; `pauseTick` tolerates it, so each tick succeeds and the check never runs.

`test-budget.yaml`'s `internal/orchengine` row rises from 142 to 143.

**ImpactSummary:** One new optional seam on the orch watcher, wired only by `orch watch`; a watcher with no check, as in every existing test, runs exactly as before.

**Commit:** 6: orch watch: re-exec onto a replaced binary at a restart-safe tick
