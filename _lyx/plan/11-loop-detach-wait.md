# Card 11 — loop-detach-wait

**Create:**
- `plan:internal/shedverbs#LoopPIDRecord` -> `type LoopPIDRecord struct`
- `plan:internal/shedverbs#ReadLoopPIDRecord` -> `func ReadLoopPIDRecord(path string) (LoopPIDRecord, bool, error)`
- `plan:internal/shedverbs#runUntilStop` -> `func runUntilStop(ctx context.Context, spec *Spec, argv []string, out io.Writer) int`
- `plan:internal/shedverbs#spawnLoop` -> `func spawnLoop(spec *Spec, argv []string) (int, error)`
- `plan:internal/shedverbs#waitLoopEnvelope` -> `func waitLoopEnvelope(ctx context.Context, spec *Spec) ([]byte, bool, error)`
- `plan:internal/shedverbs#envelopeCurrent` -> `func envelopeCurrent(envelope []byte, st shedengine.Status) bool`
- `plan:internal/shedverbs#deadLoopStop` -> `func deadLoopStop(ctx context.Context, spec *Spec, rec LoopPIDRecord, out io.Writer) int`
- `plan:internal/shedverbs#procChildRunner` -> `func procChildRunner(ctx context.Context, req ChildRequest) (Child, error)`
- `plan:internal/shedverbs#loopDetachedFlag` -> `const loopDetachedFlag = "loop-detached"`
- `plan:internal/shedverbs#waitPollFloor` -> `const waitPollFloor = time.Second`
- `plan:internal/shedverbs#waitPollCeiling` -> `const waitPollCeiling = 30 * time.Second`
- `internal/shedverbs/loopwait_test.go`
- `internal/shedverbs/loop_integration_test.go`
- `internal/shedverbs/testmain_integration_test.go`

**Edit:**
- `internal/shedverbs#stepCmd`
- `plan:internal/shedverbs#runLoop`
- `plan:internal/shedverbs#LoopSpec`
- `internal/shedverbs#shedverbsAllowedImports`
- `internal/shedverbs/doc.go`
- `cmd/lyx#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`
- `pattern/PATTERN-shed-verb-set.md`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/proc#StartTied`
- `plan:internal/proc#KillTree`
- `plan:internal/proc#SelfRecord`
- `plan:internal/proc#HoldNamedJob`
- `plan:internal/proc#IsBreakawayRefused`
- `internal/fswatch#Watch`
- `internal/lock#TryAcquireWriteLock`
- `plan:internal/shedengine#WriteFailedStop`

**Intent:** D2: the loop runs detached and the driver's command is a waiter on it, with the loop lock as the liveness record, the pid file for the kill, the Windows named job as the loop's identity, and the child tie from `internal/proc`.
`stepCmd`'s `--until-stop` now calls `runUntilStop`; a hidden flag, `loopDetachedFlag`, marks the detached loop process itself, which runs `runLoop`.
`runUntilStop` first reads the loop lock and the pid file: a held loop lock means a live loop, so it spawns nothing and waits; a free lock with a pid file and no envelope marks a loop that died without one, so it runs `deadLoopStop`; a free lock with an envelope delivers it when `envelopeCurrent` says its `loop.current_producer`, `loop.history_length` and `loop.state` still match the status file, else retires it undelivered; otherwise it spawns a loop through `spawnLoop` and waits.
`spawnLoop` runs `spec.Loop.Executable` with the loop's argv plus `loopDetachedFlag`, stdout and stderr appended to `spec.Loop.LogPath`, configured by `proc.DetachBreakaway`; a start error `proc.IsBreakawayRefused` recognises is retried with `proc.Detach` and logged, since such a loop lives only as long as the shell's job.
The spawn is logged with the pid.
`waitLoopEnvelope` waits for the envelope file: through `fswatch.Watch` on the steps directory where it opens, else by polling, never below `waitPollFloor`, doubling to `waitPollCeiling` while nothing changes and returning to the floor on a change; it ends when the envelope appears, or when the loop lock is free with no envelope (the loop is gone), which the waiter reports as a dead loop through `deadLoopStop` when a pid file is present and as `interrupted` with cause `loop-exited` otherwise.
A waiter prints the envelope and retires it, so the next invocation spawns a fresh loop; a waiter killed before the envelope arrives loses nothing.
`runLoop` gains the loop's own life: it takes the loop lock through `lock.TryAcquireWriteLock` and exits at once, before any step and without an envelope, when the lock is held; it holds the named job through `proc.HoldNamedJob(spec.Loop.JobName)` for its life; it writes the pid file after taking the lock, with `proc.SelfRecord` and, per child, the in-flight child's `TreeRecord` plus the status file's `current_producer` and `history_length` at the child's start; it removes the pid file only after its envelope is on disk.
`LoopPIDRecord` is that file's shape, exported with `ReadLoopPIDRecord` for the teardown of card 17, and the loop writes it through one `state`-style atomic write.
`procChildRunner` is the production `ChildRunner`: `proc.StartTied` over an `exec.Cmd` built from the request, so a child dies with the loop and never with the driver's shell, and each child's `Record` is what the pid file carries.
`deadLoopStop` first kills what is left of the dead loop's step tree through `proc.KillTree` over the recorded child record, then writes `failed` through `shedengine.WriteFailedStop` with cause `loop-exited`; it fills `loop.status_moved` from the recorded producer and history length against the status file, writes the `interrupted` envelope with `interrupt_policy` of the file's `current_producer` row as the loop's envelope file, retires the pid file and prints the stop; when the write meets another holder of the run lock the stop ends `busy`, keeps the pid file, writes no loop envelope and prints the `busy` stop, so the next invocation retries.
An invocation never spawns a loop over a dead loop's pid file.
`LoopSpec` gains the seams the tier-1 tests need: `Spawn func(argv []string) (int, error)` (nil selects `spawnLoop`), `Watch func(dir string, names ...string) (<-chan struct{}, func(), error)` (nil selects `fswatch`), and `Sleep func(context.Context, time.Duration) error`.
`shedverbsAllowedImports` gains `internal/lock` and `internal/fswatch`; `PATTERN-shed-verb-set.md` states the allowlist as it now stands.
`cmd/lyx#unboundedWaitFiles` lists `internal/shedverbs/loopwait.go`, and `PATTERN-daemon-wakeups.md` lists the `--until-stop` waiter among the unbounded waits, with `fswatch` as its event source.
`refusal-spec.md` gains the shed rows `loop exited` (the `loop-exited` interrupted stop, way forward: the driver's interrupted rule under the row's policy) and `loop busy at a dead-loop stop` (transient, way forward: re-run `lyx shed step <run-id> --until-stop`).
`doc.go` describes the waiter, the liveness record and the currency rule.
`loopwait_test.go`, tier 1 over a fake `Spawn` that runs `runLoop` in a goroutine with the fake runner of card 10 and a fake `Watch`: `TestUntilStop_SecondWaiterAdoptsTheLiveLoop` (a second `--until-stop` spawns nothing and prints the same envelope); `TestUntilStop_EnvelopeCurrency`, a table over a current unwatched envelope delivered then retired, a stale one retired and a fresh loop spawned, and a halt envelope stale once the file reads `running` again; `TestUntilStop_DeadLoopStopsLoopExited`, with the busy variant keeping the pid file and the next invocation retrying; `TestUntilStop_LoserWaitsOnWinner`, two loops spawned at once where only the lock holder steps and the loser's waiter writes no `failed`.
`loop_integration_test.go` (`integration` tag) runs real children through `procChildRunner` against a `lyx` built by `internal/testkit/lyxbin` in a temporary worktree: a `lyx shed step <run-id>` child over an unseeded run exits without a record and stops the loop as `interrupted` with cause `exited`, a captured stderr file and a trace file carrying the child's own id, distinct from the loop's; and, on non-Windows, a child whose tree holds a grandchild dies with the loop and survives the waiter's death.
`testmain_integration_test.go` runs that binary under `gitkit.HermeticGitEnv` and `tmuxkit.Main`, as `internal/shedcli`'s does.
`test-budget.yaml` raises `internal/shedverbs` from 74 to 80.

**ImpactSummary:** `stepCmd`'s `--until-stop` changes from foreground to detached, with no caller outside this package; `LoopSpec` gains seams its filler may leave nil; the allowlist gains two imports.
