# Card 11 — loop-body

**Create:**
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#ChildRequest`
- `internal/shedverbs#Child`
- `internal/shedverbs#ChildRunner`
- `internal/shedverbs#LoopStop`
- `internal/shedverbs#LoopStopHalted`
- `internal/shedverbs#LoopStopCondition`
- `internal/shedverbs#LoopStopError`
- `internal/shedverbs#LoopStopInterrupted`
- `internal/shedverbs#LoopStopBusy`
- `internal/shedverbs#loopObject`
- `internal/shedverbs#loopRestep`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#stopDetail`
- `internal/shedverbs#loopEnvelope`
- `internal/shedverbs#interruptedRecord`
- `internal/shedverbs#interruptedError`
- `internal/shedverbs#stdoutErrEnvelope`
- `internal/shedverbs#detailLineBudget`
- `internal/shedverbs#detailByteBudget`
- `internal/shedverbs#UntilStopFlag`
- `internal/shedverbs#ArmStop`
- `internal/shedverbs#ReportLoopArmError`
- `internal/shedverbs/loop_test.go`

**Edit:**
- `internal/shedverbs#Spec`
- `internal/shedverbs#Hooks`
- `internal/shedverbs#stepCmd`
- `internal/shedverbs#shedverbsAllowedImports`
- `internal/shedverbs/doc.go`
- `internal/shedcli#shedCLI.resolvePersistentPreRun`
- `internal/shedcli/cli_test.go`
- `internal/loomcli#loomCLI.resolvePersistentPreRun`
- `internal/loomcli#loomCLI.arm`
- `pattern/PATTERN-shed-verb-set.md`
- `PATTERN.md`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shedengine#WriteFailedStop`
- `internal/shedverbs#ReportArmError`
- `internal/shedrun#LoopLock`
- `internal/shedrun#StatusFile`
- `internal/shedrun#RunLock`
- `internal/shedrun#StatusLock`
- `internal/logger#NewTraceID`
- `internal/proc#TreeRecord`
- `internal/shedverbs#KindInterrupted`
- `internal/shedverbs#shortStepEnvelope`
- `internal/shedverbs#shortStepErrFields`
- `internal/shedverbs#stepErrFields`

**Intent:** The loop mode of D1, with D5's failure write, D7's rich stop answer and D8's transient re-step, as a body in `loop.go` and `loopenvelope.go` beside `stepCmd`, driven through seams so it is tested at tier 1 with fakes.
`Spec` gains `Loop LoopSpec`; `LoopSpec` carries the told loop paths `LockPath`, `PIDPath`, `LogPath`, `EnvelopePath` and `JobName`, `StopFiles func(traceID string) (traceCopy, stderr string)`, `TraceFiles func(traceID string) ([]string, error)`, `Runner ChildRunner`, `Executable string`, `Now func() time.Time`, and `IdleTimeout` and `Activity` for card 13; a zero `EnvelopePath` means the recipe arms no loop.
`TraceFiles` names a child's trace files, told by the arming module (card 15), because `PATTERN-shed-verb-set` lets `shedverbs` report `logger.TraceFile` and `logger.TraceDir` only verbatim and never join, read or copy under them; the loop reads and copies only the paths `TraceFiles` and `StopFiles` tell it.
`Child` is `Wait() error`, `Quit() error`, `Kill() error` and `Record() proc.TreeRecord`; `ChildRequest` carries the argv, the environment, the working directory, a `Stdout io.Writer` and the stderr path.
`stepCmd` registers `--until-stop` under the name `UntilStopFlag`; in this card the flag runs `runLoop` in the foreground, and card 12 moves it behind the detached spawn.
The `--until-stop` branch returns before `rec.begin()`, so the invocation carrying the flag writes no in-flight record of its own and `lyx shed status`'s `last_step` only ever names a child's step.
An arming refusal of the `--until-stop` invocation itself is a loop stop, since it refuses in the subtree's pre-run before any verb body runs: both `resolvePersistentPreRun`s, in `internal/shedcli` and `internal/loomcli`, report an arming error of a `step` whose `UntilStopFlag` is set through `ReportLoopArmError` whenever they resolved a location and a run-id, with an `ArmStop` carrying that run-id and the `StatusFile`, `RunLock`, `StatusLock` and `LoopLock` paths `internal/shedrun` constructs for it, and through card 9's `ReportArmError` otherwise.
The loom pre-run reaches its location through `c.location`: `arm` records the location `lyxcwd.Resolve` returned on the receiver before the reconcile and `armAt`, where today only the wiring sets it, at the end of a `wire` that a missing hub landing config refuses first; so a refusal from any arming stage after the resolve, the missing hub config included, finds the location set, and only a failed resolve, which names no run, takes `ReportArmError`.
`ReportLoopArmError` keeps a `KindlessRefusal` kind-less exactly as `ReportArmError` does; for any other error it composes the same `bootstrap` message, way-forward clause and logged trace, then, when the status file exists and the loop lock is free, calls `shedengine.WriteFailedStop` with that text, and prints the `bootstrap` error envelope plus the `loop` object with `steps` 0, `stop` `error`, `detail` the error text, `status_moved` false and `current_producer`, `history_length` and `state` read from the status file, the other keys empty.
Bound: a held loop lock means a live loop owns the run, so the refusal writes nothing and the stop is `busy`, as it is when `WriteFailedStop` reports `Busy`; an absent status file writes nothing and leaves the status keys empty.
A `--until-stop` on a spec whose `Loop.EnvelopePath` is empty is refused on a `bootstrap` error envelope: `shedverbs: this recipe arms no loop; way forward: run the step without --until-stop`.
`runLoop` iterates: it mints a trace id with `logger.NewTraceID`, runs one child through `spec.Loop.Runner` with the loop's own argv minus `--until-stop` (and the flag of card 12), from the same cwd, with `LYX_TRACE_ID` set to the fresh id in the environment, stdout into a buffer the loop reads after the child exits and then appends to the loop log, and stderr to the `StopFiles` stderr path for that id; after the child exits it reads the child's own record, `<StepsDir>/<traceID>.json`, as the step's full envelope.
A child that left no record but printed an error envelope on stdout refused before its step body ran, as a pre-step arming refusal of card 9 does: `stdoutErrEnvelope` decodes the last non-empty stdout line, and an `ok: false` envelope is taken as the stop step's error envelope, its missing short-form keys filled empty, so the stop is `error` with the child's own `kind` (`bootstrap` for an arming refusal) and `trace_file`, never `interrupted`.
It stops at: a record whose `continue` is false, a record or stdout envelope in the error form, a child that left neither (interrupted), and a stop condition that fired, seen as `paused` with a reason from the engine's condition pause.
The stop cause is `halted` for `continue` false on `blocked`, `awaiting`, `done` or a bare-pause `paused`, `stop-condition` for a `paused` whose reason is a condition reason, `error` for an error record or stdout envelope, `interrupted` for neither, and `busy` for a `busy` error or a busy failed-stop write.
An error with a non-empty `transient` gets one immediate re-step, recorded as `loopRestep` (the producer, the first failure's error and class); a failing re-step stops with the second failure as the stop step's envelope, and a successful step resets the count, so a later transient failure at the same producer gets its own re-step.
On an `error` or `interrupted` stop while the status file still reads `running`, the loop calls `shedengine.WriteFailedStop` with the stop's error text, its class and its way-forward clause; a `Busy` result turns the stop into `busy`.
An interrupted stop's error text comes from `interruptedError`, which returns the message and its way-forward clause: `shedverbs: step <producer> interrupted (<cause>); way forward: read loop.trace_copy and loop.stderr_path, apply the interrupted rule under loop.interrupt_policy, then re-run lyx shed step <run-id> --until-stop`, with `<producer>` the status file's `current_producer`, so the envelope, the status file's `error` and the refusal row all name the way forward (`PATTERN-refusal-way-forward`).
An interrupted stop fills the short error form itself: the error is that text, `kind` is `KindInterrupted`, `transient` empty, `run_id` the run's, `trace_file` the first of the child's `TraceFiles`, or empty when it has none, and `friction` is `Hooks.AfterInterrupt`'s return, the hook this card adds to `Hooks`, `func(ctx context.Context, producer, cause, traceFile string) string`, nil-skipped like the others; `interruptedRecord` writes the full error form, with the five location keys, as `<traceID>.json` in the steps directory and returns the record and its path for `envelope_path`.
The loop's final envelope is `loopEnvelope`: the stop step's short envelope, success or error form through `shortStepEnvelope` or `shortStepErrFields`, plus one `loop` object whose key set is closed at `steps`, `first_producer`, `stop`, `detail`, `status_moved`, `restep` (omitted when none ran), `current_producer`, `history_length`, `state`, `interrupt_policy`, `trace_copy` and `stderr_path`; `current_producer`, `history_length` and `state` are read from the status file at the stop, `status_moved` compares the file's producer and history length before and after the stop step, and `interrupt_policy` is `Hooks.InterruptPolicyFor` over the stop row.
`stopDetail` composes `detail` for an error, interrupted or halt stop from the told trace files: the step's error or reason, the transient class, and the WARN and ERROR lines of the stop step's trace plus its last lines, capped at `detailLineBudget` lines and `detailByteBudget` bytes; a successful iteration prints nothing, and the final `done` envelope is the short success envelope plus `loop`.
After the child has exited the loop copies the stop step's trace files, in `TraceFiles` order, into the `StopFiles` trace-copy path, so a sweep of the trace directory leaves the copy, and names both files in `loop`.
The loop writes the final envelope to `spec.Loop.EnvelopePath` and prints it; the exit code is the stop step's.
The loop stops on its own when the status file vanishes, reporting `interrupted` with cause `status-missing`.
Every child spawn, with its pid and trace id, and every stop, with its cause, is logged through `internal/logger`.
`shedverbsAllowedImports` gains `internal/proc` for the record type; `PATTERN-shed-verb-set.md` states the grown allowlist, the loop body's home, and that a child's trace files reach the loop told through `Spec.Loop`, while `logger.TraceFile` and `logger.TraceDir` stay reported verbatim.
`PATTERN.md`'s shed-verb-set line states the allowlist in the same commit, becoming "Adding a run, step, status, pause or goto verb: the generic bodies live in `internal/shedverbs` only, which derives no path, imports only its allowlist and closes the step kinds at six. (test)" with its background link, about 15902 bytes under the Shared Decision `pattern-overview-cap`; the allowlist's members are listed in the background file, which card 12 updates again.
`refusal-spec.md`'s shed section gains the rows `loop unarmed` (correctness halt) and `step interrupted` (the loop's `interrupted` stop, class transient, with the way forward `interruptedError` writes).
`docs/overview.md`'s loom `step` sentence names `--until-stop`.
`doc.go` describes the loop, its stop causes and the `loop` key set.
`loop_test.go` drives the loop through `stepCmd` with `--until-stop` and `execEnvelope`, over a spec whose `LoopSpec` carries a fake `ChildRunner` that writes step records, stdout envelopes and status files the way a child would and a fake `TraceFiles` over files in the test's directory: `TestLoop_RunsUntilStopAndPrintsOneEnvelope`, a table over halted (blocked, awaiting, done, bare pause), stop-condition, busy, and a child's arming refusal on stdout stopping as `error` with kind `bootstrap`, asserting the step keys equal the stop step's short envelope, the `loop` key set is closed, and the steps directory holds no in-flight record under the invoking process's own trace id; `TestLoop_InterruptedChildFillsTheErrorForm`, asserting kind, cause, the way-forward clause in the error and in the status file, the written record, `envelope_path`, the failed status write and the hook's friction; `TestLoop_TransientRestepOnce`, covering a passing re-step, a failing re-step with both failures reported, and a re-step after an intervening success; `TestLoop_FailureStopLeavesStatusFailedUnlessBusy`, the failed write and its busy exception; `TestLoop_StopDetailAndTraceCopy`, the detail budget and the copy surviving a removed original; `TestLoop_UnarmedSpecRefused`.
`internal/shedcli/cli_test.go`'s `TestArmFromSeed_RefusalEnvelopeKinds` gains rows rendering an unknown recipe on `step` through `ReportLoopArmError` over an `ArmStop` in the test's directory: a `running` status file is written `failed` with the refusal and the envelope carries `kind: bootstrap` and the closed `loop` key set with `stop` `error` and `steps` 0; a held run lock and a held loop lock each leave the file untouched with `stop` `busy`; an absent status file writes nothing; a missing run stays kind-less with no `loop` object.
`test-budget.yaml` raises `internal/shedverbs` from 66 to 72.

**ImpactSummary:** `Spec` and `Hooks` gain fields their fillers may leave zero; `stepCmd` gains one flag whose absence leaves today's path byte-identical; the allowlist gains one import; loom's `arm` sets `c.location` earlier, which every later reader already expects set once `armAt` returns.
