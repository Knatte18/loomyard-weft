# Card 11 — dobby-pass-act

**Edit:**
- `internal/dobbyengine#RunView`
- `internal/dobbyengine#dobbyengineAllowedImports`
- `internal/dobbyengine#drivable`
- `internal/dobbyengine#planPass` -> `func planPass(run RunView, rs RunState, trigger passTrigger) passKind`
- `internal/dobbyengine#daemon.drivePasses`
- `internal/dobbyengine#runLabel`
- `internal/dobbyengine#daemon.heartbeat`
- `internal/dobbyengine#strandedEnded`
- `internal/dobbyengine#daemon.readWaits`
- `internal/dobbyengine#daemon.runNotices`
- `internal/dobbyengine#TestPlanPass`
- `internal/dobbyengine#TestDrivePasses_ExitHandling`
- `internal/dobbyengine/doc.go`
- `internal/battencli#DobbyRuns`
- `internal/battencli#DobbyRunCells.Of`
- `internal/battencli#TestDobbyRunCells_ReadsTheRunsFromDisk`
- `internal/beamcli#TestDobbyRuns_PairAtItsOwnAnchor`
- `internal/beamcli#TestDobbyRuns_ReadsWhetherAPausedRunsWorkReachedItsParent`
- `internal/beamcli#TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitAndTheEndedEntryAndPair`
- `internal/beamcli#TestDobbyLandings_CountsOnlyThisMachine`

**Uses:**
- `plan:internal/pairteardown#ReadRun`
- `plan:internal/runfsm#Judge`
- `plan:internal/runfsm#TransitionState`

**Intent:** The dobby's pass asks the act rule first and reads each run's state from the FSM (D1).
- `RunView` gains `Run runfsm.State` and `Verdict runfsm.Verdict`; its `CancelMarker` doc says it reads a cancel under way: an unfinished cancel intent in the record, or a cancel lock another process holds (card 8).
  So `dobbyengine` imports `internal/runfsm`, and `dobbyengineAllowedImports` gains `github.com/Knatte18/loomyard/internal/runfsm`, which `TestToldGeometryInvariant_AllowlistOnly` otherwise fails.
  `DobbyRuns` fills both per run through `pairteardown.ReadRun` and `runfsm.Judge` with nothing set aside; a read that fails leaves the run out of this tick with one Warn per episode, as an unreadable status does today.
- `planPass` gives no pass unless the run is `drivable`, `run.CancelMarker` is false and `Verdict.Answer` is act; the act rule takes the place of its `Held` check alone.
  `run.CancelMarker` stays in its first gate beside `drivable`:
  `runfsm.Judge` reads the batten record alone, so in the window between the cancel lock's take and the intent's write it answers act, and only `CancelMarker` sees the held lock (card 8).
  `drivable` stays as the first gate, so a run batten halted (blocked, failed or awaiting) gets no pass however the rule answers, as today: the act rule says whether this machine may act on a run, and `drivable` whether a pass has anything to do.
  `drivable` drops its paused-with-work-on-parent clause and reads only running, since the act rule waits every batten-paused run (Shared Decisions: paused-before-landing), so that clause can no longer give a pass.
  `strandedEnded` keeps that clause itself: a paused run whose work reached its parent is not stranded, so the dobby never tears down a pair whose Land is unfinished.
  `RunView.ReachedParent` and `DobbyRuns`' reached-parent read of a paused run stay for it.
  `drivable`'s doc reads running only; `strandedEnded`'s doc and `doc.go`'s stranded-run paragraph say such a paused run is not stranded because its Land is unfinished, no longer because it gets a pass;
  `planPass`'s doc drops the paused-with-work-on-parent pass, names the act verdict where it names the adopt pause, and names the cancel marker as a cancel under way.
  A cancelling run, a paused or covered one and one owned elsewhere on the local Bolt all read wait or hand over.
  Its `covered` parameter goes, since the rule already waits a covered run; the continue pass under a pause goes with it, and the full pass at a lift stays.
  The remembered owned-elsewhere branch from a pass exit stays as it is.
- `drivePasses` drops the pending lift of a run when `!drivable(run) || run.CancelMarker || verdict is not act`, where it reads `drivable` and the marker today, and calls `planPass` without `covered`.
- `runLabel` and the heartbeat name the FSM state, the producer beside it as today; the done-run filter and the done-run clear keep reading batten's own `done`, which D3 keeps as today.
- The comments of `strandedEnded`, `daemon.readWaits` and `daemon.runNotices` that name the cancel marker name a cancel under way: an unfinished cancel intent, or a cancel lock another process holds.
- `DobbyRunCells.Of`, the overview's run cells, names its state cell from `runfsm.TransitionState` over batten's state and stage, the word the board shows, so the overview reads the FSM's vocabulary.
- `doc.go`'s pass-planning paragraph says the act rule decides beside a cancel under way, which gives no pass,
  and that a paused run whose work reached its parent gets no pass and waits until `lym batten run <slug>` resumes it;
  every other paragraph that names the cancel marker names a cancel under way: an unfinished cancel intent in the batten record, or a cancel lock another process holds.

Tests:
- `TestPlanPass` (tier0) rewrites its covered rows: a run whose verdict waits or hands over gets none on every trigger, an act verdict gets the passes the triggers give today, and a blocked run with an act verdict gets none; the lift still gives a full pass.
  A row keeps `CancelMarker` true with an act verdict, and it gets none on every trigger.
  Its paused-with-work-on-parent rows give none.
- `TestDrivePasses_ExitHandling` (tier0) rewrites its subtest of a lift seen by a paused run whose work reached its parent: the run gets no pass after the lift.
- `TestDobbyRunCells_ReadsTheRunsFromDisk` (tier0) asserts the FSM words in its state cells.
- The four `beamcli` tests that read `DobbyRuns` and `DobbyLandings` over a hub (tier1) run `ReadRun` over that hub now;
  `TestDobbyRuns_ReadsTheLandOutcomeTheLandingCommitAndTheEndedEntryAndPair` gains assertions of `Run` and `Verdict` for a running run, a paused one and one with an unfinished cancel intent.

**ImpactSummary:** The pass planner's gate gains the act rule and loses its covered parameter, both callers in drivePasses updated; the run view grows two fields battencli fills, and the overview's state cell takes the FSM's words.
