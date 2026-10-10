# Card 12 — loop-detach-wait

**Create:**
- `plan:internal/shedverbs#LoopPIDRecord` -> `type LoopPIDRecord struct`
- `plan:internal/shedverbs#ReadLoopPIDRecord` -> `func ReadLoopPIDRecord(path string) (LoopPIDRecord, bool, error)`
- `plan:internal/shedverbs#WriteLoopPIDRecord` -> `func WriteLoopPIDRecord(path string, rec LoopPIDRecord) error`
- `plan:internal/shedverbs#runUntilStop` -> `func runUntilStop(ctx context.Context, spec *Spec, argv []string, out io.Writer) int`
- `plan:internal/shedverbs#spawnLoop` -> `func spawnLoop(spec *Spec, argv []string) (<-chan struct{}, error)`
- `plan:internal/shedverbs#waitLoopEnvelope` -> `func waitLoopEnvelope(ctx context.Context, spec *Spec, spawned <-chan struct{}) ([]byte, bool, error)`
- `plan:internal/shedverbs#envelopeCurrent` -> `func envelopeCurrent(envelope []byte, st shedengine.Status) bool`
- `plan:internal/shedverbs#deadLoopStop` -> `func deadLoopStop(ctx context.Context, spec *Spec, rec LoopPIDRecord, out io.Writer) int`
- `plan:internal/shedverbs#procChildRunner` -> `func procChildRunner(ctx context.Context, req ChildRequest) (Child, error)`
- `plan:internal/shedverbs#LoopDetachedFlag` -> `const LoopDetachedFlag = "loop-detached"`
- `plan:internal/shedverbs#TeardownLoopID` -> `const TeardownLoopID = "teardown"`
- `plan:internal/shedverbs#loopLockGrace` -> `const loopLockGrace = 5 * time.Second`
- `plan:internal/shedverbs#waitPollFloor` -> `const waitPollFloor = time.Second`
- `plan:internal/shedverbs#waitPollCeiling` -> `const waitPollCeiling = 30 * time.Second`
- `internal/shedverbs/loopwait_test.go`
- `internal/shedverbs/loop_integration_test.go`
- `internal/shedverbs/testmain_integration_test.go`
- `pattern/PATTERN-failure-stop-visible.md`

**Edit:**
- `internal/shedverbs#stepCmd`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#ArmStop`
- `internal/shedverbs#ReportLoopArmError`
- `internal/shedverbs#shedverbsAllowedImports`
- `internal/shedcli#shedCLI.resolvePersistentPreRun`
- `internal/loomcli#loomCLI.resolvePersistentPreRun`
- `internal/shedverbs/doc.go`
- `cmd/lyx#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`
- `pattern/PATTERN-shed-verb-set.md`
- `PATTERN.md`
- `contracts/specs/refusal-spec.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/proc#StartTied`
- `internal/proc#KillTree`
- `internal/proc#SelfRecord`
- `internal/proc#HoldNamedJob`
- `internal/proc#IsBreakawayRefused`
- `internal/shedverbs#interruptedError`
- `internal/logger#NewTraceID`
- `internal/fswatch#Watch`
- `internal/lock#TryAcquireWriteLock`
- `internal/lock#AcquireWriteLockWithin`
- `internal/shedengine#WriteFailedStop`
- `internal/shedrun#LoopPIDFile`
- `internal/shedrun#LoopEnvelope`

**Intent:** D2: the loop runs detached and the driver's command is a waiter on it, with the loop lock as the liveness record, the pid file for the kill, the Windows named job as the loop's identity, and the child tie from `internal/proc`; and D5's invariant, `PATTERN-failure-stop-visible`, lands here, where its waiter and its dead-loop stop are built.
`stepCmd`'s `--until-stop` now calls `runUntilStop`; a hidden string flag, `LoopDetachedFlag`, exported for the two pre-runs, marks the detached loop process itself, which runs `runLoop`, and carries its loop id, which the spawning waiter mints with `logger.NewTraceID`; like card 11's branch, the detached branch returns before `rec.begin()`.
Bound on that flag: it only selects `runLoop` in the process `spawnLoop` starts, and `runLoop` itself refuses to step over a dead loop's record, so a hand-typed `--loop-detached` can skip `runUntilStop`'s checks but can never re-step a half-run producer or overwrite a dead loop's pid file: after taking the loop lock, a `runLoop` that finds the teardown's mark, or a pid file and no envelope, logs the refusal and exits without stepping and without an envelope, leaving the record for the next `--until-stop`, whose waiter reports the dead loop.
`runUntilStop` first reads the loop lock and the pid file: a free lock beside the teardown's mark, a pid file whose loop id is `TeardownLoopID` (card 18), runs `deadLoopStop` whatever else the directory holds; a held loop lock with a pid file means a live loop, so it spawns nothing, adopts the loop id the pid file records and waits; a held loop lock with no pid file is either an instant liveness probe or a loop between taking its lock and writing its pid file, so it adopts nothing and re-reads on the wait timer until a pid file appears, which it adopts, or the lock reads free, when it takes this decision again; a free lock with a pid file and no envelope marks a loop that died without one, so it runs `deadLoopStop`; a free lock with an envelope delivers it when `envelopeCurrent` says its `loop.current_producer`, `loop.history_length` and `loop.state` still match the status file, else retires it undelivered, by the same rename onto its loop's delivered record with nothing printed; otherwise it spawns a loop through `spawnLoop` and waits.
`spawnLoop` runs `spec.Loop.Executable` with the argv `runUntilStop` built, the loop's own plus `LoopDetachedFlag` set to the minted loop id, which the spawning waiter adopts, stdout and stderr appended to `spec.Loop.LogPath`, configured by `proc.DetachBreakaway`; a start error `proc.IsBreakawayRefused` recognises is retried with `proc.Detach` and logged, since such a loop lives only as long as the shell's job.
The spawn is logged with the pid, and `spawnLoop` returns a channel closed when the spawned process exits, which it reaps.
Start handshake: the waiter that spawned a loop treats it as starting until that channel closes, so a free lock seen before the new loop took it never reads as a gone loop; a waiter that found the lock held already waits on a loop that has started.
`runLoop` takes the loop lock through `lock.AcquireWriteLockWithin(spec.Loop.LockPath, loopLockGrace)`, so a liveness probe that holds the lock for an instant, from a second waiter or from card 18's quiet probe, never makes a starting loop exit; a loop that still cannot take the lock after `loopLockGrace` exits at once, before any step and without an envelope, and its waiter, seeing its spawn exited and the lock held, adopts the holder's loop id from the pid file and waits on it.
`waitLoopEnvelope` waits for the envelope file: it re-checks the envelope, the loop lock and the spawned channel on a timer that starts at `waitPollFloor`, doubles to `waitPollCeiling` while nothing changes and returns to the floor on a change, and where `fswatch.Watch` on the steps directory opens, an event only cuts the current wait short, so a loop that dies without touching a file is still seen at the next tick.
The loop envelope file holds the loop's id beside the envelope, as `loop_id` and `envelope`, and a waiter prints only `envelope`.
A waiter matches files by its adopted loop id: it prints the envelope file only when its `loop_id` equals that id, or the delivered record at `spec.Loop.Delivered(id)`, which only that loop's envelope ever reaches, so it never prints another loop's envelope; a delivered record for its id was delivered or retired by another invocation, so it prints that record's envelope too, retires nothing and writes nothing.
It ends when either file carries its id, or when its loop is gone.
Since a loop writes its envelope before removing its pid file and every retirement lands in that loop's own delivered record, a loop that finished always leaves one of the two files carrying its id, so the rules below see only a loop that never finished.
A free lock beside the teardown's mark ends every waiter through `deadLoopStop`, whatever id it adopted.
While the pid file names its id, its loop is live with the loop lock held and gone with the lock free, which `deadLoopStop` reports; while its own spawn has not exited, its loop is starting.
Otherwise what the waiter saw decides:
a waiter whose own spawn exited, with the lock held, adopts the loop the pid file names, as the loser rule above says, and re-reads while no pid file is there;
a waiter whose own spawn exited before any pid file named it, with the lock free, saw a loop that died before stepping, so it reports `interrupted` with cause `loop-exited` and `detail` naming `spec.Loop.LogPath`, which holds the dead process's output, writes `failed` through `WriteFailedStop` under its bound while the status file reads `running`, and prints the stop without writing a loop envelope.
A waiter only ever reaches the gone rule with an adopted loop id, its own spawn's or a pid file's, so an instant probe's hold never reads as a loop that existed and then went.
The detached process runs its subtree's pre-run again before `runLoop`, so an arming input that changed since the waiter's own pre-run armed can refuse there: both pre-runs fill `ArmStop.LoopID` from `LoopDetachedFlag`, and with a loop id set `ReportLoopArmError` writes its envelope, under that `loop_id`, as the loop envelope file at `ArmStop.EnvelopePath` instead of printing it into the loop log, so the waiter prints the `bootstrap` stop as its loop's envelope; with the loop lock held it writes nothing, as its bound already says, and the waiter adopts the holder.
`ArmStop` also gains `PIDPath`, which both pre-runs fill from `shedrun.LoopPIDFile`, as they fill `EnvelopePath` from `shedrun.LoopEnvelope`.
With the loop lock free and a pid file present, `ReportLoopArmError` keeps D2's order: it first kills the dead loop's recorded step tree through `proc.KillTree` and logs the kill, then writes `failed`, and leaves the pid file, so the next invocation's `deadLoopStop` still reports the dead loop as `loop-exited`.
A waiter prints the envelope and retires it by renaming the file onto `spec.Loop.Delivered` of the envelope's `loop_id`, so the next invocation spawns a fresh loop while a second waiter on the same loop still prints it, and a later loop's delivery lands in its own record; a waiter killed before the envelope arrives loses nothing.
Once the loop lock is free, a waiter or invocation that delivers or retires an envelope also retires a pid file left beside it whose loop id equals the envelope's `loop_id`, since the envelope proves that loop finished, and never the teardown's mark, so a loop that died between its envelope and its pid-file removal never reads as a dead loop later.
`runLoop` gains the loop's own life: it holds the named job through `proc.HoldNamedJob(spec.Loop.JobName)` for its life; it writes the pid file after taking the lock, with its loop id, `proc.SelfRecord` and, per child, the in-flight child's `TreeRecord` plus the status file's `current_producer` and `history_length` at the child's start; it removes the pid file only after its envelope is on disk.
`LoopPIDRecord` is that file's shape, exported with `ReadLoopPIDRecord` and `WriteLoopPIDRecord`, its one `state`-style atomic write, for the teardown of card 18.
`procChildRunner` is the production `ChildRunner`: `proc.StartTied` over an `exec.Cmd` built from the request, so a child dies with the loop and never with the driver's shell, and each child's `Record` is what the pid file carries.
`deadLoopStop` first kills what is left of the dead loop's step tree through `proc.KillTree` over the recorded child record and logs the group, the pid and whether anything was killed, then writes `failed` through `shedengine.WriteFailedStop` with the text and way-forward clause `interruptedError` returns for cause `loop-exited`; it fills `loop.status_moved` from the recorded producer and history length against the status file, writes the `interrupted` envelope with `interrupt_policy` of the file's `current_producer` row as the loop's envelope file under the dead loop's recorded id, retires the pid file and prints the stop.
A record carrying no child, such as a loop's before its first child or the teardown's mark of card 18, kills nothing, since a zero group id must never reach a kill, and reports `status_moved` false.
Over the teardown's mark, `deadLoopStop` writes `failed` under the same bound and prints the `loop-exited` stop with `loop.detail` naming the teardown, but writes no loop envelope and keeps the mark, so every invocation and waiter until the pair's removal deletes the worktree stops the same way and spawns nothing.
Bound: only card 18's seize writes the mark, and it puts back the pid file the mark replaced when the session end fails, so the mark bars loops only in a pair whose session has ended.
When `WriteFailedStop` reports the file not `running`, as after an arming refusal wrote `failed` over the dead loop, the stop writes no status and goes on as written.
When that write meets another holder of the run lock the stop ends `busy`, keeps the pid file, writes no loop envelope and prints the `busy` stop with the error `shedverbs: the run lock is held, so the dead loop of run <run-id> is not yet stopped; way forward: re-run lyx shed step <run-id> --until-stop once the holder ends`, so the next invocation retries.
An invocation never spawns a loop over a dead loop's pid file.
`LoopSpec` gains `Delivered func(loopID string) string`, the told delivered-record path of a loop id, and the seams the tier-1 tests need: `Spawn func(argv []string) (<-chan struct{}, error)` (nil selects `spawnLoop`), `Watch func(dir string, names ...string) (<-chan struct{}, func(), error)` (nil selects `fswatch`), and `Sleep func(context.Context, time.Duration) error`.
`shedverbsAllowedImports` gains `internal/lock` and `internal/fswatch`; `PATTERN-shed-verb-set.md` states the allowlist as it now stands.
`cmd/lyx#unboundedWaitFiles` lists `internal/shedverbs/loopwait.go`, and `PATTERN-daemon-wakeups.md` lists the `--until-stop` waiter among the unbounded waits, with `fswatch` as its event source and the timer as its floor.
`PATTERN.md` gains, under "Shed and loom", the line "`PATTERN-failure-stop-visible` — Stopping a `--until-stop` loop on a failure with no other run-lock holder: the status file is non-running before its waiter returns. (test)" with its background link, in the entry shape `internal/pattern`'s doc prescribes, leaving the overview at about 16137 bytes under the Shared Decision `pattern-overview-cap`; the new background file states the invariant, that it covers a loop that died without an envelope and an arming refusal of the `--until-stop` invocation itself (card 11's `ReportLoopArmError`), that `lyx loom resume` therefore never answers "already running" for such a stopped run, that no failure waits for the quiet notice, that the Go driver and a bare `step` stay outside it, and that `TestLoop_FailureStopLeavesStatusFailedUnlessBusy`, `TestUntilStop_DeadLoopStopsLoopExited` and `TestLoopIntegration_ArmingRefusalStopsAsBootstrap` enforce it.
`refusal-spec.md` gains the shed rows `loop exited` (the `loop-exited` interrupted stop, with the way forward `interruptedError` writes) and `loop busy at a dead-loop stop` (transient, way forward: re-run `lyx shed step <run-id> --until-stop` once the holder ends).
`doc.go` describes the waiter, the start handshake, the loop id and its adoption, the liveness record, the per-loop delivered records, the teardown's mark, the detached flag's bound and the currency rule.
`loopwait_test.go` drives `stepCmd` with `--until-stop` through `execEnvelope`, over a fake `Spawn` that runs the detached half (`stepCmd` with `--loop-detached`) in a goroutine with the fake runner of card 11 and closes its channel when that returns, a fake `Watch` and a fake `Sleep`: `TestUntilStop_SecondWaiterAdoptsTheLiveLoop` (a second `--until-stop` spawns nothing and prints the same envelope, with a row where both waiters are alive when the envelope lands and both print it, neither reporting `loop-exited` nor writing `failed`, and rows where the first waiter sleeps through its loop's delivery and a later loop's run, one where it had read its loop's pid file and one where it never had, so in both it prints its own loop's envelope from that loop's delivered record and neither the later loop's envelope, nor `loop-exited`, nor a `failed` write); `TestUntilStop_EnvelopeCurrency`, a table over a current unwatched envelope delivered then retired with a leftover pid file, a stale one retired and a fresh loop spawned, and a halt envelope stale once the file reads `running` again; `TestUntilStop_DeadLoopStopsLoopExited`, a table over a dead loop found at start, a loop that dies while the fake watch emits nothing, the busy variant keeping the pid file and the next invocation retrying, a `--loop-detached` run over a dead loop's pid file stepping nothing, a record with no child killing nothing and stopping `loop-exited`, the teardown's mark stopping a first, a second and a third invocation `loop-exited`, each spawning nothing, writing no loop envelope and keeping the mark, with a variant where an undelivered envelope lies beside the mark, a waiter that read a held lock with no pid file stopping `loop-exited` once the lock is released beside the teardown's mark, an arming refusal through `ReportLoopArmError` over a dead loop's pid file writing `failed` and keeping the record, after which the next invocation stops `loop-exited` with no second status write, a spawned loop whose own pre-run refuses through `ReportLoopArmError` with a loop id, whose waiter prints that `bootstrap` envelope, and a spawned loop that exits before any pid file names it, whose waiter stops `loop-exited` with `detail` naming the loop log and the file written `failed`; `TestUntilStop_LoserWaitsOnWinner`, two loops spawned at once where only the lock holder steps and the loser's waiter writes no `failed`, with a row where the waiter polls before its loop takes the lock and keeps waiting, and a row where the first read sees a loop lock the test holds for an instant with no loop behind it, and the invocation spawns a loop once the lock reads free and writes no `failed`.
`loop_integration_test.go` (`integration` tag) runs real children through `procChildRunner` against a `lyx` built by `internal/testkit/lyxbin` in a temporary worktree: `TestLoopIntegration_ArmingRefusalStopsAsBootstrap`, a table over a loom-seeded run whose hub landing config is absent and whose status file reads `running`: in one row a `lyx shed step <run-id>` child refuses while arming, and the loop stops as `error` with kind `bootstrap`, a captured stderr file and a trace file holding the refusal and carrying the child's own id, distinct from the loop's; in the other the built binary runs `lyx shed step <run-id> --until-stop` itself, the surface the driver runs, and its own pre-run's refusal answers with kind `bootstrap`, the closed `loop` key set with `stop` `error`, and the status file reading `failed`; a third row runs `lyx loom step --until-stop` over the same fixture, the loom surface, and asserts the same answer and the same `failed` write; and, on non-Windows, `TestLoopIntegration_ChildTreeDiesWithLoop`, a child whose tree holds a grandchild dies with the loop and survives the waiter's death.
`testmain_integration_test.go` runs that binary under `gitkit.HermeticGitEnv` and `tmuxkit.Main`, as `internal/shedcli`'s does.
`test-budget.yaml` raises `internal/shedverbs` from 72 to 78.

**ImpactSummary:** `stepCmd`'s `--until-stop` changes from foreground to detached, with no caller outside this package; `LoopSpec` gains seams its filler may leave nil; the allowlist gains two imports; `ArmStop` gains three fields both pre-runs fill, and `ReportLoopArmError` gains the dead-tree kill and the detached envelope write, its envelope unchanged otherwise; one PATTERN line and its background are added.

**Verify:** GOOS=windows go vet ./internal/proc ./internal/shedverbs
