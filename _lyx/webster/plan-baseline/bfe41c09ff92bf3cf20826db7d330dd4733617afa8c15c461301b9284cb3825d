# Card 8 — shed-yield

**Edit:**
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#Shed.Run`
- `internal/shedengine#RunOutcome`
- `internal/shedengine/doc.go`

**Create:**
- `plan:internal/shedengine#Yield` -> `const Yield Outcome = "yield"`
- `plan:internal/shedengine#RunYielded` -> `const RunYielded RunOutcome = "yielded"`
- `plan:internal/shedengine#TestStep_YieldPersistsNothing` -> `func TestStep_YieldPersistsNothing(t *testing.T)`

**Intent:**
Add `Yield`, the shed outcome of a producer at a wait its own next call re-checks (D4).
`stepLocked` gains a `Yield` arm before the default: it appends no history entry, persists nothing (no status write, no activity, no commit-status call) and returns `StepResult{Producer, Outcome: Yield, Next: current_producer, State: StateRunning, History}`; no bounce budget, reroute or route is read.
`Run` returns on that step with `Result{Outcome: RunYielded, HaltedProducer: res.Next, History}` instead of continuing, so the step-until-stop loop ends; every other running step still continues.
`RunYielded` joins `RunOutcome`'s values, the one value with no `State` twin, and the doc comments of `Outcome` and `RunOutcome` say so.
Bound: a yield leaves the run exactly as it was, at the same row and `running`; the doc states that only a producer at a wait its own next call re-checks returns it, and that InnerRun is the only one.
The package doc's outcome list gains `Yield`.
`TestStep_YieldPersistsNothing` (tier0, parallel) over `newTestShed`: a producer returning `Yield` leaves the status file's bytes and modification time unchanged, appends no history, calls no `CommitStatus`, and `Step` reports `Outcome: Yield, State: running, Next` the same row; a second row runs `Run` and gets `RunYielded` after one call, with the producer called once.

**ImpactSummary:** One new arm in stepLocked and one in Run; every existing outcome path is untouched, and the unrecognised-outcome arm still refuses anything else.
