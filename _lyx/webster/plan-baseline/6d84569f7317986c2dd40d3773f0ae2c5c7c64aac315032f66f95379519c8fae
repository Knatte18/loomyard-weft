# Card 3 — gate-hold-terminal

**Edit:**
- `internal/shuttleengine/gate.go`
- `internal/shuttleengine/wait.go`
- `internal/shuttleengine/doc.go`
- `internal/shedadapters/singlellm.go`
- `internal/shedadapters/gateattempts.go`
- `internal/shedadapters/singlellm_test.go`

**Create:**
- `internal/shuttleengine/gate_terminal_test.go`

**Intent:**
Give the gate engine the two capabilities Decision 5 needs from it, without naming any entry.

`GateEntry` gains a bool field, `MayHold`, that lifts `evaluateGate`'s refusal of `Pending` and of a non-nil `Final` from an entry that is not `PassOnCap`.
The field is the only lift: the two existing guards become "not `PassOnCap` and not `MayHold`".
The lift lets nothing through.
A `MayHold` entry stays must-pass: a failure sets `outcome.Passed` false, and a `Pending` result at a final evaluation (`evaluateGate(true)`) counts as not passed, so finalize never reads a held must-pass entry as a pass.
A `Pending` at a turn boundary holds the run exactly as a `PassOnCap` entry's does today.
Document on the field that only the parent-review wiring sets it.

`GateResult` gains `Terminal bool`: a failing result the run finalizes at once, with no re-prompt, whatever the entry's in-memory failure count.
`Terminal` on a passed or pending result is a returned gate error from `evaluateGate`, like the existing contract violations.
`evaluateGate` records whether the stopping failure was terminal, and `handleGatedBoundary` finalizes on it beside the existing spent-budget test.
The in-memory count is not incremented for a terminal failure.

`GateOutcome` gains `Reason string`: the terminal failing result's `Findings` text, empty otherwise.
It exists because finalize deletes the run directory and with it the findings file, so a terminal entry's explanation has to ride the outcome to the producer.
`gateFailedReason` in `gateattempts.go` returns `gate.Reason` when it is non-empty and keeps its current text otherwise, so `SingleLLMProducer.mapOutcome`'s Stuck carries the terminal entry's own explanation; that Stuck becomes the run's `blocked` reason when the row has no `on_stuck`.
Check `BurlerProducer`'s gate-failed exit, which also calls `gateFailedReason`, still reads correctly with the change.

Update the doc comments of `GateResult`, `GateEntry`, `PassOnCap`, `GateEntryWaiting`, `GateOutcome`, `handleGatedBoundary`, `evaluateGate`, and the gate paragraph of `internal/shuttleengine/doc.go` that says only `PassOnCap` entries may pend.
The Completion Signal Invariant's tripwire (`completionsignal_enforcement_test.go`) must stay green: the terminal path finalizes `OutcomeDone` through the existing call, adding no negative-verdict site.

Tests, in `gate_terminal_test.go` following `gate_pending_test.go`'s fakes: a `MayHold` must-pass entry may return pending and set `Final`; a non-`MayHold`, non-`PassOnCap` entry doing either is still a gate error; a terminal failure with budget left finalizes on that arrival without sending a re-prompt and carries `Reason`; a non-terminal failure below budget still re-prompts; `Terminal` on a passed result is a gate error; a `MayHold` entry whose `Final` returns pending fails the outcome.
In `singlellm_test.go`, a failing gate outcome carrying `Reason` maps to Stuck with that reason, and one without keeps `gateFailedReason`'s text.

**ImpactSummary:** Adds opt-in gate fields (`MayHold`, `Terminal`, `Reason`); every existing entry leaves them zero and keeps its current behavior.
