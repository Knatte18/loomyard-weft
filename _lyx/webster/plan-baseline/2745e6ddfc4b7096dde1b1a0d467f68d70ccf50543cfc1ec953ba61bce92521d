# Card 6 — driver-start-check

**Create:**
- `plan:internal/sheddriver#goDriver.checkBlockedStart` -> `func (d *goDriver) checkBlockedStart(status shedengine.Status, statusFound bool) (handedBack bool, err error)`

**Edit:**
- `internal/sheddriver#Deps`
- `internal/sheddriver#goDriver.run`
- `internal/sheddriver#goDriver.resumeOpenAttempt` -> `func (d *goDriver) resumeOpenAttempt(ctx context.Context, status shedengine.Status, statusFound bool) (stop Stop, found bool, err error)`
- `internal/sheddriver#TestGoDriver_Settle`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/sheddriver/doc.go`
- `internal/loomcli/doc.go`

**Uses:**
- `plan:internal/loomcli#loomCLI.finalizeGuard`

**Intent:**
D5: a Go driver that starts at a blocked stop checks the current row before its first step, so a hand-over successor, or a driver a bare `lyx loom resume` spawned, never re-runs Finalize into the same dirty-parent refusal and writes no second halt note.

- `sheddriver.Deps` gains an optional `StartCheck func(row string) (refusal string, err error)`; nil checks nothing, so a recipe that wires none steps as today.
- `goDriver.run` makes the driver's first status read itself, before anything else, and hands back through `handBackFailure` with `reading the status before the first step` when it fails, the text `resumeOpenAttempt` returns today.
  It hands that one read to `resumeOpenAttempt`, which no longer reads the status itself, and, when no open attempt is found, to `checkBlockedStart` before its first `Step`, so both decide on the same first read D5 names.
- `checkBlockedStart` runs `StartCheck(status.CurrentProducer)` only for a found status in state `blocked`.
  A refusal hands back at the stop built from that status (`stopFromStatus`) through `handBackAs`, with the refusal text as the hand-back's rule and `the start check refused the blocked stop` as its outcome, so the stop report quotes the text, the run is parked and nothing is stepped.
  A `StartCheck` error hands back through `handBackFailure`, naming the check.
  An admit, or a status that is not blocked, steps as today.
  A driver resuming an open attempt skips the check, since that attempt's verdict is acted on first.
- `loomCLI.goDriverDepsFor` sets `StartCheck: c.finalizeGuard()`: D4's Finalize arm alone, never the settled-segment arm, so a blocked stop at any other row steps as today, and the Finalize row is admitted when the parent pair cannot be opened or holds a fabric merge record.
  The one-line assignment itself has no Tier-1 test, since `goDriverDepsFor` resolves the arbiter and tackler config and the on-disk executable; card 5's wiring rows pin that `c.finalizeGuard()` refuses a dirty `Finalize` and admits a settled review row, so the arm it names is the Finalize arm alone.
- `internal/sheddriver/doc.go`'s list of optional seams gains `no StartCheck: a driver starting at a blocked stop steps it unchecked`, and its loop description says what the check does.
- `internal/loomcli/doc.go`'s hand-over sentence, which says the fresh `lyx loom drive` re-steps, and its sentence on the driver `lyx loom resume` spawns for a halted run, say that such a driver starting at a blocked Finalize stop first runs the Finalize arm and hands back, stepping nothing, while the parent's code checkout is dirty.

Bound: the check only reads, runs once per driver before the first step, and can only hand back; it never moves the run.
The rule table is unchanged, because its only re-step without a goto is the interrupted-step rule, never a blocked stop.

Tests, rows of `TestGoDriver_Settle`, whose harness gains `startCheck bool`, `startRefusal string` and `startErr error`: when `startCheck` is set the harness wires `StartCheck`, which appends `start-check:<row>` to the calls and returns the two values.
- A blocked initial status at `Build` and a refusal: calls `start-check:Build` only, one hand-back record, one stop report holding the refusal, parked, exit hand-back; no step runs, so no step writes a halt note.
- A blocked initial status and an admit, then a `done` step: calls `start-check:Build`, `step`, `commit-records`, exit done.
- A blocked initial status and a check error: no step, one hand-back, the report holding the error text.
- An awaiting initial status with the check wired and a `done` step: calls `step`, `commit-records` with no `start-check`.
- No status file with the check wired and a `done` step: calls `step`, `commit-records` with no `start-check`.
- The existing open-attempt restart row with the check wired refusing: the attempt is resumed and no `start-check` call is made.

**ImpactSummary:** one optional seam on `sheddriver.Deps` read at a driver's start, wired only by `loomcli`'s driver deps, and `resumeOpenAttempt`'s one caller, `goDriver.run`, now hands it the first status read; every existing row runs with the seam nil and keeps its calls.
