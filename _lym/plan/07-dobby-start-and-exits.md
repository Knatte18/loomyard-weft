# Card 7 — dobby-start-and-exits

**Edit:**
- `internal/dobbyengine#State`
- `internal/dobbyengine#Deps`
- `internal/dobbyengine#Run`
- `internal/dobbyengine#daemon`
- `internal/dobbyengine#daemon.recordStart`
- `internal/dobbyengine#daemon.loop`
- `internal/dobbyengine#daemon.beamOwnedElsewhere`
- `internal/dobbyengine#daemon.tick`
- `internal/dobbyengine#newLoopHarness`
- `internal/dobbyengine#TestRun_LockOwnerAndWakeups`
- `internal/beamcli#hubDobbyDeps`
- `internal/beamcli#runDobby`
- `internal/dobbyengine/doc.go`

**Create:**
- `plan:internal/dobbyengine#reexecLockWait` -> `const reexecLockWait = 10 * time.Second`
- `plan:internal/dobbyengine#ReadsAsReexec` -> `func ReadsAsReexec(statePath string, pid int, startTicks string) (bool, error)`
- `plan:internal/dobbyengine#TestReadsAsReexec` -> `func TestReadsAsReexec(t *testing.T)`

**Extends:**
- `internal/beamcli/doc.go`

**Uses:**
- `plan:internal/selfexec#Watcher.CheckWith`
- `plan:internal/dobbyengine#AppendLog`
- `internal/beamcli#registerDobby`
- `internal/dobbyengine#loopHarness.run`

**Intent:**
Decisions 1, 2 and 3 inside `Run`: the lock is the one decision whether this process is the dobby, a re-exec waits longer for it, and every way the loop ends or fails to re-exec leaves a line, the abnormal ones a notice.

State and the re-exec test:
- `State` gains `StartTicks string` with the tag `json:"start_ticks,omitempty"`, the dobby's own `proc.StartTime`, which an exec keeps; tolerant (PATTERN-tolerant-records).
- `ReadsAsReexec` first stats `statePath`'s directory and reports false when it does not exist, creating nothing, since `ReadState` goes through `state.ReadJSON`, which creates the directory (card 10 runs the test on worktrees that may hold none); otherwise it reads the record through `ReadState` and reports true when it is found, holds `pid`, and holds `startTicks` or no start ticks. A record without ticks was written by a binary from before this change, so the first re-exec onto this binary is that case. Its doc comment carries Decision 2's bound: a fresh dobby that reused such a record's pid after a reboot reads as a re-exec.
- `Deps` gains `StartTicks func() string`, nil reading `proc.StartTime(os.Getpid())` (an unreadable value is empty); `ReexecLockWait time.Duration`, zero meaning `reexecLockWait`; and `Locked func() error`, run once the lock is held and before the state record is written, nil doing nothing, which the CLI fills with the run-process registration (Decision 1). `CheckBinary` becomes `func(beforeExec func(), failed func(error))`, the shape of `selfexec.Watcher.CheckWith`. Struct fields take no re-sign arrow.

`Run`, in order:
1. Runs `ReadsAsReexec` on `StatePath` with this pid and `StartTicks`; a read error is a fatal error whose test could not run: it appends `dobby: stopped on error: <error>` to `LogPath` with `rotate` false, posts nothing, and `Run` returns the error.
2. Waits `lock.ProbeGrace` for the lock on a start that fails the test and the re-exec wait on one that passes it.
3. When the lock stays held, appends `dobby: start refused: another dobby holds the lock` to `LogPath` with `rotate` false, since this process holds no lock. On a re-exec it reads the record again: when it still holds this pid it posts `<worktree>: the dobby could not take its lock after a re-exec; way forward: run lym beam status, and once it shows the dobby not alive, run lym beam start`, since `lym beam start` reads a held lock as a live dobby and starts nothing while the holder keeps it, else it posts nothing, since another dobby runs. A fresh refused start posts nothing (the idempotent case `EnsureLive` relies on). It returns `ErrDobbyRunning`; `Locked` never ran, so no run-process record is written.
4. Holding the lock, calls `Locked`, then `recordStart`, then logs `start fresh pid <pid>` or `start re-exec pid <pid>`.
5. A non-nil error out of `Run` after step 1, other than `ErrDobbyRunning`, logs `dobby: stopped on error: <error>` (rotating only while the lock is held) and, when the start passed the re-exec test, posts `<worktree>: the dobby stopped on an error after a re-exec: <error>; way forward: fix the cause, then run lym beam start` through `Deps.Notice`.

`recordStart` uses the same test in place of the pid-only one: a record with this pid and these ticks owes nothing; one with this pid and no ticks owes a full pass and gains this process's ticks, every other field kept; any other start owes a full pass and rewrites pid, start time and start ticks.
`tick`'s refill of a record found without a pid writes the start ticks beside the pid (Decision 2: every writer of `State.PID` writes the ticks).

`loop` logs `stop: another machine owns this beam (<owner>); lym adopt <subject> takes it over here` on the owner exit, the subject being `root` for the prime and the beam's slug otherwise, as `EnsureLive`'s refusal names them, and `stop: the context ended` when a cancelled context ends it, and calls `CheckBinary` with two callbacks: one logging `re-exec onto the changed lym binary` right before the exec, and one logging `re-exec failed: <error>; the dobby keeps running` and posting `<worktree>: the dobby could not re-exec onto the changed lym binary: <error>; it keeps running on the old binary; way forward: fix the binary, then run lym beam stop and lym beam start`. A failed post is logged through `logger.Warn`. `selfexec` runs the failure callback once per identity, which bounds the notice.

`beamcli`: `runDobby` no longer calls `dobbyengine.Live` before `Run`, and no longer calls `registerDobby` before `hubDobbyDeps`; it sets the returned `Deps.Locked` to the `registerDobby` call. `hubDobbyDeps` wires `CheckBinary` to `selfexec.New().CheckWith`.
Bound (Decision 1): `hubDobbyDeps` still runs before `Run`, so a fresh dobby is unregistered from its spawn until it holds the lock, at most the wiring time plus `lock.ProbeGrace`, and a beam dobby spawned inside a teardown of its own beam escapes that teardown; card 8 ends such a dobby once its worktree is gone.

Tests (tier0; the lock waits are real flock waits, kept short through `ReexecLockWait`):
- `newLoopHarness` fills `StartTicks` with a fixed value, a two-callback `CheckBinary`, a `Log` that records each call's path, line and `rotate` flag, a recording `Notice`, and a recording `Locked`.
- `TestRun_LockOwnerAndWakeups` gains or extends rows: a held lock on a fresh start returns `ErrDobbyRunning`, logs the refused line with `rotate` false, posts nothing and never calls `Locked`; a re-exec record whose lock stays held past a short `ReexecLockWait` logs the refused line and posts one notice naming `lym beam start`; the same under a one-second `ReexecLockWait` whose record a goroutine rewrites to another pid 500 ms into the wait, after the first read has passed the re-exec test, posts nothing; a re-exec record whose lock the test releases after `lock.ProbeGrace` plus one second, under a three-second `ReexecLockWait`, a margin wide enough for a loaded gate, takes the lock and ticks; a record with this pid and other ticks under the same release is refused after `lock.ProbeGrace` and posts nothing, and with a free lock owes a full pass and is left holding this process's ticks; a record with this pid and no ticks under the same release takes the lock, owes a full pass and is left holding the ticks, and under a held lock posts the refused-re-exec notice; a `CheckBinary` fake that calls its `failed` callback once, on its first call only, as `selfexec` does once per identity, writes the failed line and one notice and the loop reaches `stopAt`; a `Locked` error after a re-exec returns the error, logs the stop line and posts one notice, and after a fresh start posts none; a fresh start with a free lock logs `start fresh pid <pid>`; a state path whose directory holds a directory named `state.json` makes the re-exec read fail, and `Run` returns the error, logs the stop line with `rotate` false and posts nothing; the owner-exit row asserts its stop line names `lym adopt root` and the cancel row asserts its stop line; the own-pid row writes this process's ticks first and the undecodable-record row asserts the refilled record holds the ticks.
- `TestReadsAsReexec` (tier0, temp record): own pid with own ticks and with no ticks read true; own pid with other ticks, another pid and no record read false; a state path whose directory does not exist reads false and the directory is still absent afterwards.

Docs: `internal/dobbyengine/doc.go`'s loop section gains the re-exec test, the two lock waits, the `Locked` seam and each line and notice; `internal/beamcli/doc.go` gains that the registration runs only once the lock is held.

**ImpactSummary:** `Run`'s start path and three seams change for the one production wiring in `hubDobbyDeps`/`runDobby` and the loop test harness; the state record gains one tolerant field.

**Commit:** 7: Take the dobby lock first, wait longer after a re-exec, and log every way a dobby ends
