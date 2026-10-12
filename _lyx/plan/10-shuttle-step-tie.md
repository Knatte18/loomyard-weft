# Card 10 — shuttle-step-tie

**Create:**
- `internal/shuttleengine#StepTie`
- `internal/shuttleengine#StepStopReport`
- `internal/shuttleengine#Runner.SetStepTie`
- `internal/shuttleengine#Runner.StopStepRuns`
- `internal/shuttleengine#TestRunner_StopStepRuns_StopsOnlyTheKilledStepsLiveRuns`

**Edit:**
- `internal/shuttleengine#RunState`
- `internal/shuttleengine#Runner`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#TestRunner_Start_HappyPath`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
Tie each shuttle run to the shed step that started it, and give the arming module a stop verb over one step's runs (D10, issue #567).

- `StepTie` is `StepsDir` (the shed run's steps directory, `shedrun.StepsDir`) and `TraceID` (the step's trace id, which names its record `<StepsDir>/<TraceID>.json`).
- `RunState` gains `Step *StepTie`, JSON `step`, omitted when nil.
- `Runner.SetStepTie` tells the runner the step its process runs; `start` copies it into the record it first persists. A runner never told one, the loom driver's and the orch's among them, writes no step.
- `StopStepRuns(tie)` scans the runner's run-directory root: a record still reading running whose strand reed reports live is stopped through `StopStrand` when its `Step` equals `tie`, which records `stopped` and removes the strand (`PATTERN-shuttle-stop`), and is listed under `Untied` without being touched when it carries no step; a run of another step is left alone.
  `StepStopReport` holds `Stopped` and `Untied` as strand names, and `Failed` as strand name and error text for a stop that failed, which skips that run and goes on.
  A record that does not decode is skipped as the attach scan skips it.

Bound: the stop takes only runs whose record names `tie`, never a run with no step and never another step's.

Tests: `TestRunner_Start_HappyPath` asserts a runner told a tie persists it in `run.json` and one told none persists no `step` key.
The new test seeds a run-directory root over the fake reed with four running records, one tied to the step, one tied to another step, one untied and one tied but terminal, and asserts the report stops exactly the first (its record reads `stopped`, its strand removed), lists the untied one, and leaves the other two and their strands as they were.
`doc.go` names the step tie and the stop verb; the budget row grows by one.

**ImpactSummary:** RunState gains an optional field and Runner a told tie and a stop verb over its own run directories; runners never told a tie write records exactly as today.
