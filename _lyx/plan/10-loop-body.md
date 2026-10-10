# Card 10 — loop-body

**Create:**
- `plan:internal/shedverbs#LoopSpec` -> `type LoopSpec struct`
- `plan:internal/shedverbs#ChildRequest` -> `type ChildRequest struct`
- `plan:internal/shedverbs#Child` -> `type Child interface`
- `plan:internal/shedverbs#ChildRunner` -> `type ChildRunner func(ctx context.Context, req ChildRequest) (Child, error)`
- `plan:internal/shedverbs#LoopStop` -> `type LoopStop string`
- `plan:internal/shedverbs#LoopStopHalted` -> `const LoopStopHalted LoopStop = "halted"`
- `plan:internal/shedverbs#LoopStopCondition` -> `const LoopStopCondition LoopStop = "stop-condition"`
- `plan:internal/shedverbs#LoopStopError` -> `const LoopStopError LoopStop = "error"`
- `plan:internal/shedverbs#LoopStopInterrupted` -> `const LoopStopInterrupted LoopStop = "interrupted"`
- `plan:internal/shedverbs#LoopStopBusy` -> `const LoopStopBusy LoopStop = "busy"`
- `plan:internal/shedverbs#loopObject` -> `type loopObject struct`
- `plan:internal/shedverbs#loopRestep` -> `type loopRestep struct`
- `plan:internal/shedverbs#runLoop` -> `func runLoop(ctx context.Context, spec *Spec, argv []string, out io.Writer) int`
- `plan:internal/shedverbs#stopDetail` -> `func stopDetail(stepErr, reason, transient, traceFile string) string`
- `plan:internal/shedverbs#loopEnvelope` -> `func loopEnvelope(step map[string]any, loop loopObject) map[string]any`
- `plan:internal/shedverbs#interruptedRecord` -> `func interruptedRecord(spec *Spec, traceID, cause, friction string) (map[string]any, string)`
- `plan:internal/shedverbs#detailLineBudget` -> `const detailLineBudget = 40`
- `plan:internal/shedverbs#detailByteBudget` -> `const detailByteBudget = 8192`
- `internal/shedverbs/loop_test.go`
- `pattern/PATTERN-failure-stop-visible.md`

**Edit:**
- `internal/shedverbs#Spec`
- `internal/shedverbs#Hooks`
- `internal/shedverbs#stepCmd`
- `internal/shedverbs#shedverbsAllowedImports`
- `internal/shedverbs/doc.go`
- `pattern/PATTERN-shed-verb-set.md`
- `PATTERN.md`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/shedengine#WriteFailedStop`
- `plan:internal/logger#NewTraceID`
- `plan:internal/proc#TreeRecord`
- `plan:internal/shedverbs#KindInterrupted`
- `internal/shedverbs#shortStepEnvelope`
- `internal/shedverbs#shortStepErrFields`
- `internal/shedverbs#stepErrFields`

**Intent:** The loop mode of D1, with D5's failure write, D7's rich stop answer and D8's transient re-step, as a body in `loop.go` and `loopenvelope.go` beside `stepCmd`, driven through seams so it is tested at tier 1 with fakes.
`Spec` gains `Loop LoopSpec`; `LoopSpec` carries the told loop paths `LockPath`, `PIDPath`, `LogPath`, `EnvelopePath` and `JobName`, `StopFiles func(traceID string) (traceCopy, stderr string)`, `Runner ChildRunner`, `Executable string`, `Now func() time.Time`, and `IdleTimeout` and `Activity` for card 12; a zero `EnvelopePath` means the recipe arms no loop.
`Child` is `Wait() error`, `Quit() error`, `Kill() error` and `Record() proc.TreeRecord`; `ChildRequest` carries the argv, the environment, the working directory and the stdout and stderr paths.
`stepCmd` registers `--until-stop`; in this card the flag runs `runLoop` in the foreground, and card 11 moves it behind the detached spawn.
A `--until-stop` on a spec whose `Loop.EnvelopePath` is empty is refused on a `bootstrap` error envelope: `shedverbs: this recipe arms no loop; way forward: run the step without --until-stop`.
`runLoop` iterates: it mints a trace id with `logger.NewTraceID`, runs one child through `spec.Loop.Runner` with the loop's own argv minus `--until-stop` (and the flag of card 11), from the same cwd, with `LYX_TRACE_ID` set to the fresh id in the environment, stdout to the loop log and stderr to the `StopFiles` stderr path for that id; after the child exits it reads the child's own record, `<StepsDir>/<traceID>.json`, as the step's full envelope.
It stops at: a record whose `continue` is false, a record in the error form, a child that left no record (interrupted), and a stop condition that fired, seen as `paused` with a reason from the engine's condition pause.
The stop cause is `halted` for `continue` false on `blocked`, `awaiting`, `done` or a bare-pause `paused`, `stop-condition` for a `paused` whose reason is a condition reason, `error` for an error record, `interrupted` for no record, and `busy` for a `busy` error record or a busy failed-stop write.
An error record with a non-empty `transient` gets one immediate re-step, recorded as `loopRestep` (the producer, the first failure's error and class); a failing re-step stops with the second failure as the stop step's envelope, and a successful step resets the count, so a later transient failure at the same producer gets its own re-step.
On an `error` or `interrupted` stop while the status file still reads `running`, the loop calls `shedengine.WriteFailedStop` with the stop's error text and class; a `Busy` result turns the stop into `busy`.
An interrupted stop fills the short error form itself: the error is the cause, `kind` is `KindInterrupted`, `transient` empty, `run_id` the run's, `trace_file` the child's trace file (`logger.TraceDir()` joined with the trace file name for that id, or empty when the sink resolves none), and `friction` is `Hooks.AfterInterrupt`'s return, the hook this card adds to `Hooks`, `func(ctx context.Context, producer, cause, traceFile string) string`, nil-skipped like the others; `interruptedRecord` writes the full error form, with the five location keys, as `<traceID>.json` in the steps directory and returns the record and its path for `envelope_path`.
The loop's final envelope is `loopEnvelope`: the stop step's short envelope, success or error form through `shortStepEnvelope` or `shortStepErrFields`, plus one `loop` object whose key set is closed at `steps`, `first_producer`, `stop`, `detail`, `status_moved`, `restep` (omitted when none ran), `current_producer`, `history_length`, `state`, `interrupt_policy`, `trace_copy` and `stderr_path`; `current_producer`, `history_length` and `state` are read from the status file at the stop, `status_moved` compares the file's producer and history length before and after the stop step, and `interrupt_policy` is `Hooks.InterruptPolicyFor` over the stop row.
`stopDetail` composes `detail` for an error, interrupted or halt stop: the step's error or reason, the transient class, and the WARN and ERROR lines of the stop step's trace plus its last lines, capped at `detailLineBudget` lines and `detailByteBudget` bytes; a successful iteration prints nothing, and the final `done` envelope is the short success envelope plus `loop`.
After the child has exited the loop copies the stop step's trace file to the `StopFiles` trace-copy path, so a sweep of the trace directory leaves the copy, and names both files in `loop`.
The loop writes the final envelope to `spec.Loop.EnvelopePath` and prints it; the exit code is the stop step's.
The loop stops on its own when the status file vanishes, reporting `interrupted` with cause `status-missing`.
Every child spawn and every stop is logged through `internal/logger`.
`shedverbsAllowedImports` gains `internal/proc` for the record type; `PATTERN-shed-verb-set.md` states the grown allowlist and the loop body's home.
`PATTERN.md` gains, under "Shed and loom", the line "`PATTERN-failure-stop-visible` — Stopping a `--until-stop` loop on a failure with no other holder of the run lock: the status file reads non-running before the waiter returns. (test)" with its background link, and the file stays under the Shared Decision `pattern-overview-cap`; the new background file states the invariant, that `lyx loom resume` therefore never answers "already running" for such a stopped run, that no failure waits for the quiet notice, that the Go driver and a bare `step` stay outside it, and that the loop tests enforce it.
`refusal-spec.md`'s shed section gains the rows `loop unarmed` (correctness halt) and `step interrupted` (the loop's `interrupted` stop, class transient, way forward: the driver's interrupted rule under `loop.interrupt_policy`, which the envelope carries).
`docs/overview.md`'s loom `step` sentence names `--until-stop`.
`doc.go` describes the loop, its stop causes and the `loop` key set.
`loop_test.go` drives `runLoop` with a fake `ChildRunner` that writes step records and status files the way a child would: `TestLoop_RunsUntilStopAndPrintsOneEnvelope`, a table over halted (blocked, awaiting, done, bare pause), stop-condition and busy stops asserting the step keys equal the stop step's short envelope and the `loop` key set is closed; `TestLoop_InterruptedChildFillsTheErrorForm`, asserting kind, cause, the written record, `envelope_path`, the failed status write and the hook's friction; `TestLoop_TransientRestepOnce`, covering a passing re-step, a failing re-step with both failures reported, and a re-step after an intervening success; `TestLoop_FailureStopLeavesStatusFailedUnlessBusy`, the invariant and its busy exception; `TestLoop_StopDetailAndTraceCopy`, the detail budget and the copy surviving a removed original; `TestLoop_UnarmedSpecRefused`.
`test-budget.yaml` raises `internal/shedverbs` from 66 to 74.

**ImpactSummary:** `Spec` and `Hooks` gain fields their fillers may leave zero; `stepCmd` gains one flag whose absence leaves today's path byte-identical; the allowlist gains one import.
