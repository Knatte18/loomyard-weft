# Card 5 — dobby-pass-asks-act-rule

**Edit:**
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#planPass`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#TestPlanPass`
- `internal/battencli#DobbyRuns`
- `internal/battencli#TestDobbyRunCells_ReadsTheRunsFromDisk`

**Uses:**
- `internal/dobbyengine/doc.go`
- `internal/dobbyengine#drivable`
- `plan:internal/runfsm#Read`
- `plan:internal/runfsm#Act`
- `plan:internal/runfsm#Verdict.Refusal`

**Intent:**
The dobby's pass asks the act rule before it spawns a pass (D1).
`RunView` gains a `Records runfsm.Records` field; `battencli.DobbyRuns` fills it with `runfsm.Read` for each listed run, beside the fields it reads today.
`planPass` asks `runfsm.Act` first, with `Inputs.PauseCovers` from its `covered` argument unless the trigger is a lift, `Inputs.OwnedElsewhere` from `RunState.OwnedElsewhere`, and the hold left at its zero value until a later part fills it; `run.Held`, the adopt pause entry, counts as a pause that covers the run.
A `wait` returns no pass; a `hand_over` keeps today's owned-elsewhere branch, whose full pass on a lift or an uncovered probe makes the pass itself refuse and hand the run over; an `act` goes on to today's trigger logic.
So a run past its parent-side merge under a hold gets its pass, and a paused, cancelling, cancelled or landed run gets none; the `CancelMarker` and `drivable` checks the verdict now covers go, while `drivable`'s reached-parent case is what `PastParentSideMerge` answers.
A covered run no longer gets the continue pass it got while a pause covered it: the act rule waits under a pause, and the lift's full pass resumes it.
`daemon.drivePasses` logs a held run's `Verdict.Refusal` once per episode with the dobby's existing change-only log, so the operator sees the way forward.
`TestPlanPass` (tier0) rows: each `wait` reason gives no pass and names its way forward in the log, a hand-over keeps its full pass on a lift, a run past its merge under a hold gets a pass, and the rows that pinned a continue pass under a covering pause now pin none.
`TestDobbyRunCells_ReadsTheRunsFromDisk` asserts the records `DobbyRuns` fills.

**ImpactSummary:** dobby spawns no pass for a run the act rule holds; a covered run loses its continue passes until the pause lifts.

**Commit:** 5: The dobby's pass asks the act rule before it spawns
