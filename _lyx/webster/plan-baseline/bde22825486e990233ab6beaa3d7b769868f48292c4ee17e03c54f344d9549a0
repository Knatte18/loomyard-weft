# Card 4 — seat-required-advisor

**Edit:**
- `internal/seatengine#Seat`
- `internal/seatengine#Table.Validate`
- `internal/seatengine#seatSpec`
- `internal/seatengine#SeatResult`
- `internal/seatengine#seatRun`
- `internal/seatengine#seatRun.watch`
- `internal/seatengine#seatRun.result`
- `internal/seatengine#Engine.Run`
- `internal/seatengine#Engine.join`
- `internal/seatengine#Engine.Resume`
- `internal/seatengine/engine_test.go`
- `internal/seatengine/table_test.go`
- `internal/seatengine/prompt_test.go`
- `internal/seatengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/seatengine#stopSeat`
- `internal/seatengine#stopAdvisors`

**Intent:**
Add the required advisor and its acceptance hook (decisions D3, D4 and D9), so a step can depend on an advisor's accepted output before its chair goes on.

`Seat` gains `Required bool` and `Accept func(shuttleengine.Result) error`.
`Table.Validate` refuses, each as a loud error naming the seat: `Required` on the chair, `Accept` on the chair, and `Accept` on an advisor that is not `Required`.
Each is reached by a table whose only violation is that field on that seat; one `TestTable_Validate` row each.

`SeatResult` gains `Result shuttleengine.Result` (the advisor's terminal shuttle result, zero while it has none), `WaitErr error` (its wait's error), `AcceptErr error` (its `Accept`'s error) and `EndedStep bool` (true when this required advisor's end ended the step before the chair's).
`seatRun.watch` records the result and the wait error on the run, and `seatRun.result` reports them with the accept error and the ended-step flag.

A required advisor's run that ends done with its seat gate passed (no gate, or `Result.Gate.Passed`) has `Accept` called with that terminal result exactly once, in the advisor's own waiting goroutine.
Every other way a required advisor's run can end ends the step: a non-done outcome, done with its gate failed (no `Accept` call), a wait error, or an `Accept` error.
So does a required advisor that fails to start.
Ending the step, `Engine.join` stops the notice sender, stops the chair and waits for its result, stops every started advisor, the required one included (a timed-out or wait-errored run may still be live, and an advisor's pane is kept), sets `EndedStep` on that advisor, and returns the result with no failure line queued to the chair, neither the ended notice nor the not-started notice.
`Engine.Run` takes the start-failure path the same way, starting no later advisor.
A seat the engine cannot stop there is an error wrapping `ErrSeatNotStopped` with its way forward, as at every other stop.
A required advisor that ended done and was accepted stays kept until the chair ends, as every done advisor is, and the chair's end decides the step as before.
At `Engine.Resume`, a found required advisor is watched with the same `Accept` rule, and a required advisor the probe did not find ends the step at once: the chair is stopped and the advisor's result carries `EndedStep` with no terminal result.

`seatSpec` leaves `QuietHold` false on a `Required` advisor, so its held turn end still notifies the parent (D9); every other advisor stays quiet.

Bound: only a seat marked `Required` can end the step before the chair, and only through its own end or its own `Accept`; every way a required advisor can end without a successful `Accept` ends the step, so a chair never waits on output that cannot come.
The Discussion seats table marks no seat, so its runs are unchanged.

Tests:
- `engine_test.go`: a new table-driven `TestEngine_Run_RequiredAdvisorEndsTheStep` over a chair and one required advisor whose `Accept` records its calls, with rows: advisor died, timed out, wait errored, done with a failed gate (no `Accept` call), `Accept` errored, failed to start (no later advisor started), and the resume path with the required advisor not found; each row asserts the chair and the advisor were stopped, `EndedStep`, the reported result, wait error and accept error, and that no line beyond the up line reached the chair.
  A row where the advisor is done and accepted and the chair then ends asserts one `Accept` call with the done result, `EndedStep` false, and the advisor stopped only at the chair's end.
  A row with a required advisor that cannot be stopped asserts `ErrSeatNotStopped` with the way forward.
- `table_test.go`: the three refusal rows above.
- `prompt_test.go`: `TestSeatPrompts_ValuesPromptsAndSpecs` asserts a required advisor's spec has `QuietHold` false and `KeepPane` true.

Raise `internal/seatengine`'s row in `cmd/lyx/testdata/test-budget.yaml` by the new top-level test.
Describe the required advisor, its acceptance hook and its ending rule in `internal/seatengine/doc.go`'s seat model, fresh-run, notice and stop-rule sections.

**ImpactSummary:** New optional seat fields, four new SeatResult fields and an early-end branch reached only by a Required advisor; existing tables mark none.
