# Card 9 — worktree-create-precondition-halt

**Create:**
- `internal/battenshed#PreconditionError`
- `internal/battenshed#PreconditionError.Error`
- `internal/battenshed#PreconditionError.Unwrap`

**Edit:**
- `internal/battenshed#worktreeCreateProducer.Call`
- `internal/battenshed#TestWorktreeCreate_CreateErrorsStickWithTheirTextVerbatim`
- `internal/battenshed/doc.go`

**Uses:**
- `internal/shedengine#ReasonPreconditionUnmet`

**Intent:**
Let Worktree-Create halt on a precondition no tackler can settle (D11), without battenshed importing fabric: the create closure wraps such a failure in `PreconditionError`, and the producer turns it into a precondition halt.
`PreconditionError` carries `Unmet` (what is unmet, one line), `WayForward` (the command that settles it) and `Err` (the cause, returned by `Unwrap`); `Error` reads `<Unmet>; way forward: <WayForward>`.
In `worktreeCreateProducer.Call`, a `createWorktree` error that `errors.As` reads as `*PreconditionError` returns `Stuck` with `OutputPointer{Halt: true, Reason: shedengine.ReasonPreconditionUnmet + ": " + err.Error()}` and writes that reason through `reportStuck`, after the cancelled-context check as today; every other error keeps today's Stuck with its text verbatim.
`Halt` blocks the run at its own row whatever its OnStuck and spends no budget, so shed persists the reason as the blocked status's error, which the driver's precondition row (card 8) reads.
`TestWorktreeCreate_CreateErrorsStickWithTheirTextVerbatim` gains a `wantHalt` column, false on its existing rows, and rows for a `PreconditionError` returned bare and wrapped with `fmt.Errorf("…: %w", …)`: each asserts `Halt` true and a reason starting with `precondition unmet: ` that names the unmet text and the way forward.
`doc.go`'s Worktree-Create paragraph names the precondition halt.

**ImpactSummary:** Only the create producer's error branch changes, and only for the new type; batten's create closure is the type's one constructor.
