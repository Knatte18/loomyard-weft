# Card 6 — adopt-step-order

**Create:**
- `plan:internal/adoptcli#stepRun` -> `type stepRun struct`
- `plan:internal/adoptcli#runSteps` -> `func runSteps(run *stepRun, from string) error`
- `plan:internal/adoptcli#stepReconcile` -> `func stepReconcile(run *stepRun) error`
- `plan:internal/adoptcli#stepPairs` -> `func stepPairs(run *stepRun) error`
- `plan:internal/adoptcli#stepPause` -> `func stepPause(run *stepRun) error`
- `plan:internal/adoptcli#stepOwner` -> `func stepOwner(run *stepRun) error`
- `plan:internal/adoptcli#stepDeploy` -> `func stepDeploy(run *stepRun) error`
- `plan:internal/adoptcli#stepBeam` -> `func stepBeam(run *stepRun) error`
- `plan:internal/adoptcli#stepCheck` -> `func stepCheck(run *stepRun) error`
- `plan:internal/adoptcli#stepDone` -> `func stepDone(run *stepRun) error`
- `plan:internal/adoptcli#pausedRun` -> `const pausedRun = "paused by adopt"`
- `plan:internal/adoptcli#pauseTargets` -> `func pauseTargets(deps adoptDeps, here string, reconciled boardengine.Owners, tasks []boardengine.Task, runs, writing []string, rootGoing bool, outcomes map[string]*slugOutcome) ([]string, error)`
- `plan:internal/adoptcli#pauseSubject` -> `func pauseSubject(deps adoptDeps, reconciled boardengine.Owners, here, slug string, writing []string, rootGoing bool) (bool, error)`
- `plan:internal/adoptcli#pauseEntries` -> `func pauseEntries(record *pauserecord.Record, slugs, beams, listed []string, now time.Time) (paused []string, other map[string]string)`
- `plan:internal/adoptcli#TestTriesStore` -> `func TestTriesStore(t *testing.T)`

**Delete:**
- `internal/adoptcli#holdStore`
- `internal/adoptcli#holdRuns`
- `internal/adoptcli#reportHolds`
- `internal/adoptcli#holdEntries`
- `internal/adoptcli#holdTargets`
- `internal/adoptcli#holdSubject`
- `internal/adoptcli#holdsFile`
- `internal/adoptcli#holdsFileName`
- `internal/adoptcli#readHolds`
- `internal/adoptcli#appendHolds`
- `internal/adoptcli#removeHolds`
- `internal/adoptcli#liftOwnerPauses`
- `internal/adoptcli#heldRun`
- `internal/adoptcli#runningHereRun`
- `internal/adoptcli#callerLabel`
- `internal/adoptcli#world.heldSlugs`
- `internal/adoptcli#TestHoldEntries`
- `internal/adoptcli#TestHoldStore`

**Edit:**
- `internal/adoptcli#request`
- `internal/adoptcli#adoptDeps`
- `internal/adoptcli#slugOutcome`
- `internal/adoptcli#adopt`
- `internal/adoptcli#resumeSlug`
- `internal/adoptcli#runBeamSteps` -> `func runBeamSteps(deps adoptDeps, here string, owners boardengine.Owners, adopted []string, noOrch bool, result *outcome, outcomes map[string]*slugOutcome)`
- `internal/adoptcli#heldBeamWayForward` -> `func heldBeamWayForward(deps adoptDeps, here string, owners boardengine.Owners, slug string, paused bool) string`
- `internal/adoptcli#Command`
- `internal/adoptcli#checkSubjects` -> `func checkSubjects(primeRecorded bool, adopted []string) []string`
- `internal/adoptcli#deployStale`
- `internal/adoptcli#ownerWriteError`
- `internal/adoptcli#hubDeps`
- `internal/adoptcli#hubDepsAs`
- `internal/adoptcli#TestRunCheck`
- `internal/adoptcli#TestFailedEnd`
- `internal/adoptcli#world`
- `internal/adoptcli#newWorld`
- `internal/adoptcli#world.deps`
- `internal/adoptcli#TestAdopt`
- `internal/adoptcli_test#TestAdoptIntegration_SecondCloneTakesOver`

**Uses:**
- `plan:internal/adoptrecord#Record`
- `plan:internal/adoptrecord#Inputs`
- `plan:internal/adoptrecord#Steps`
- `plan:internal/adoptrecord#StateStopped`
- `plan:internal/adoptrecord#Request.Command`
- `plan:internal/pauserecord#AdoptCaller`
- `internal/orchcli#BeamUpOptions`

**Intent:**
D3, D5, D6 and D7's step side: `adopt` becomes the ordered steps of D3, each a function over one `stepRun`, with the adopt pause in place of the hold and the owner write before the deploy.
No record is persisted yet; card 7 persists the record and resumes from it.

`stepRun` holds the deps, the request, this machine's name, the resolved slugs and `rootNamed`, the owners read at the start, the outcome under construction, the per-slug outcomes, the mutation record, the origin report, and an `adoptrecord.Record` whose `Inputs` and `Steps` the steps fill.
It also holds `Save func(adoptrecord.Record) error`, called after each step ends and when a step starts; nil saves nothing.
`runSteps` runs the steps of `adoptrecord.Steps` from `from` in order.
Before the first step it seeds an outcome for each slug that the record's `Subjects` without `root`, `Inputs.Runs` and `Inputs.Paused` name and no outcome holds yet.
So a step resumed past `owner` in a fresh process never meets a nil outcome, as `runBeamSteps`'s `outcomes[slug]` would.
`root` gets no outcome, as today, and `envelopeOf` (card 7) names to `collect` only the `Subjects` without `root` that the merged parts hold,
so the envelope's `slugs` list keeps today's members.
Then, on every run, resumes included, it reads the local runs with today's `movedRuns` over the owners read at the start and runs today's standing-move refusal (`refuseStandingMove`) over the subjects and those runs.
A refusal ends the record `stopped` at `from`, with the refusal's way forward and `StoppedPart` cleared, and no step runs.
Before each step it sets `InFlight`.
After each it records a `StepRecord` with the step's end time and outcome part, as JSON.
Each step writes into a fresh outcome and fresh per-slug outcomes of its own, which are its part, and `runSteps` merges that part into the run's outcome by `envelopeOf`'s rule (card 7).
So a part holds the top-level outcome fields that step wrote, plus a `slugs` object keyed by slug holding only the `slugOutcome` fields that step set.
Its list fields (warnings, mutations, check warnings) hold only the items that step appended.
A `StepRecord` of the same name already listed, as after a restart at `reconcile`, is replaced in place, so the record lists each step once.
A step's error ends the record `stopped` at that step, with the error and its way forward, and no later step runs.
The part that step gathered before its error goes into the record's `StoppedPart`,
and `runSteps` merges it into the run's outcome as it merges a done step's part, but no `StepRecord` lists the step.
So a refused reconcile still carries the reconcile report and the mutation record, and a failed deploy its deploy outcome, as today's refusals do.
A later run of that step that ends without error clears `StoppedPart`, and a later stop replaces it.

The steps, in order (D3):
1. `stepReconcile` — first it clears every field of `Inputs` but `Paused`, so a restart never carries an earlier run's `HeldBack`, `Adoptable` or `OwnerStood`.
   Then today's reconcile and its refusals but the standing-move one, which `runSteps` runs, the moved runs before and after it (`Inputs.LocalRuns`, `Moved`), and the stop decisions of `wouldStop` with `--keep-local` and `--drop-local`.
2. `stepPairs` — today's takeover of each slug (`takeOverSlug`) and the pair add; it fills `Inputs.Runs`, `Adoptable`, `RootGoing` and `HeldBack`.
   `HeldBack` lists only the projects a local-commit or live-pair stop held back: a `wouldStop` stop, or a `takeOverSlug` outcome with `Stopped` set.
   A project held back by an error, such as a refused pair add, is held back as today, its error naming its own way forward, and joins no `HeldBack`, so `stepDone` never names `--keep-local` for it.
3. `stepPause` — D5's pause, through one `UpdatePause` applying `pauseEntries`.
   `pauseTargets` returns the slugs `holdTargets` held today, one flat sorted list:
   each moved run that went through the takeover with no stop, error or warning, and whose owner the write makes this machine's, whether or not its pair reads live here;
   and each released, unstarted task of a subject being written that is not a front task.
   The live-pair exception of today (`running here`) is dropped, since D5 pauses a run whose pair reads live here too.
   `pauseSubject` is `holdSubject` answering whether a write of this call reaches the slug.
   `pauseEntries` takes the targets, the adopted beams' own pair slugs (`beams`, never `root`) and the slugs `Inputs.Paused` already lists (`listed`).
   - It skips every listed slug, so a re-run leaves a slug the operator resumed since unpaused.
   - A slug with no own-scope entry gets a new `AdoptCaller` entry at now, its grace end at now, so it waits no grace and sends no Esc.
   - An own-scope `OwnerCaller` entry turns into an `AdoptCaller` entry in place, keeping its pause time.
   - A slug holding an entry of any other caller, legacy `adopt-hold` included, keeps it, gets none, and is returned in `other` with that caller.
   - A beam slug gets a conversion of its own-scope `OwnerCaller` entry and nothing else; a beam with none gets no entry.
     A beam whose own pawl run is among the targets follows the target rule, so the beam rule decides only an adopted beam with no moved run.
   It returns the slugs it paused, converted beams included, which join `Inputs.Paused`.
   Each paused slug's outcome reads `adopt_paused` true, `run` `paused by adopt`, and a `pause_standing` line naming `lym tender resume <slug>`.
   A failed pause update stops the record at `pause`, naming the re-run.
4. `stepOwner` — today's `SetOwners` write and the read-back, with `ownerWriteError`'s refusals.
   It fills `Inputs.OwnerStood` and `PrimeRecorded`.
   At its end, after the read-back when one runs, it calls `reconcileWarnings` over the reconcile report,
   `Inputs.OwnerStood` as the adopted beams, and the run's merged per-slug outcomes, which carry `stepPairs`' `Stopped` marks.
   The warnings join the owner step's part, so they still precede the deploy warning.
   The steps up to `owner` share one process (Shared Decisions), which is why the report and the earlier steps' outcomes are in memory there.
   `resumeSlug` keeps its reporting — the parent warning, owned elsewhere, `none` for no run record, the pause-standing line — and lifts nothing:
   `liftOwnerPauses` and the `Lift` seam go, since adopt lifts no pause (D3).
5. `stepDeploy` — `deployStale` when `Inputs.RootGoing`, as today.
   A failed deploy stops the record at `deploy`; its text says the owner stands and the adopted runs stay paused, so none goes on under the old lym (D6).
6. `stepBeam` — `runBeamSteps` for the prime when `PrimeRecorded`, and for each beam of `OwnerStood`, passing the request's `NoOrch`.
   `adoptDeps.BeamUp` becomes `func(worktree string, noOrch bool) (orchcli.BeamUpReport, error)`, and `hubDeps` sets `orchcli.BeamUpOptions.SkipOrch` from it.
   A failed beam step stays the subject's warning naming `lym beam start <subject>`.
   `heldBeamWayForward` drops "its adopt check resumes the landing here at its green end".
   It gains `paused`, which `runBeamSteps` sets when `Inputs.Paused` lists the beam's slug:
   then it names `lym tender resume <slug>` and `lym loom resume` in its pair, and otherwise `lym loom resume` in its pair alone, since no entry stands for the tender resume to lift.
7. `stepCheck` — `ResetTries` and `StartSuite` for root when `PrimeRecorded`, then each beam of `OwnerStood`, through `checkSubjects`; `ensureCheck` skips a check already running.
   The per-run report is built here, over `Inputs.LocalRuns`, `Moved`, `Paused` and the named subjects.
8. `stepDone` — the record ends `done`, or `stopped` when `Inputs.HeldBack` names a project.
   Its way forward names the re-run with `--keep-local` or `--drop-local` through `Request.Command`.

`request` gains `NoOrch`.
`adopt` validates as today, builds a `stepRun`, runs `runSteps` from `reconcile`, and returns the outcome.
A record that ends `stopped` returns an error naming its way forward, so a held-back project no longer ends the call ok.
`holdStore`, `adoptDeps.Holds`, `holdsFile` and its read and write go: `holds.json` is no longer written or read.
`slugOutcome.Held` becomes `AdoptPaused` (`adopt_paused`), and `heldRun` becomes `pausedRun`.
`slugOutcome.PauseLifted` (`pause_lifted`) goes with `liftOwnerPauses`, which was its only writer.
`runningHereRun` goes with `holdTargets`' live-pair exception, and `callerLabel` with the `Lift` seams of this card and card 4.
The comment over the run constants says a run is owned elsewhere after the owner write or paused by adopt, dropping the held and running-here outcomes.
`deployStale`'s and `ownerWriteError`'s texts drop "the holds stay" and "no run was held".
`holds.go` keeps only the tries file (`triesFile`, `triesFileName`, `readTries`, `writeTries`), and its file comment is rewritten to say so.
`Command`'s `Long` step list becomes one line naming D3's steps in order, the pause in place of the hold, and a green check that only reports; card 8 rewrites the rest of the text.
The comments that still say a green check lifts holds, starts runs or resumes the landing are reworded with the edits of their symbols:
the `adopt.go` file header;
`resumeSlug`'s doc, since it lifts nothing and starts nothing;
`runBeamSteps`' and `heldBeamWayForward`'s docs, since the landing resumes by hand with `lym loom resume`, after `lym tender resume` when adopt paused the beam;
and the `deps.go` file header, which names the adopt pause in place of the pause lift and hold, and only the tries file.

Tests:
- `TestHoldEntries` goes with no unit test of `pauseEntries`, which `TestAdopt`'s pause rows and `TestRunWorker` (card 7) reach through the steps (`PATTERN-test-speed`).
- `TestTriesStore` (tier0) keeps `TestHoldStore`'s fix-tries rows.
- `TestAdopt` (tier0): the world drops its holds files.
  The world's `Reconcile` seam appends `reconcile` to `order` and its `AddPair` appends `addpair:<slug>`, which they do not today, so a row can tell which steps ran.
  The world gains `pauseErr`, which its `UpdatePause` returns before it mutates the record.
  Its `order` shows reconcile, addpair, pause, owners, deploy, beamup and suite in D3's order, which a row pins.
  The hold rows become pause rows: an `owner` entry turned to `adopt` keeping its pause time; an `adopt-hold` and an operator entry kept; a released task paused and a front task not; a live pair under `--keep-local` paused too.
  A beam row: `alpha`'s own-scope `owner` entry is converted, and the envelope's `runs` shows `alpha` `paused_by_adopt` naming `lym tender resume alpha`.
  An adopted beam with no moved run and no own-scope entry gets none.
  A failed pause update (`pauseErr`) stops at `pause`, naming the re-run, with no `owners` in the order.
  A failed deploy row stops at `deploy`, with the owner request recorded, the pause entries standing, and the envelope still naming the deploy outcome.
  A failed `SetOwners` stops at `owner` with no deploy.
  A held-back project row ends with the error naming the re-run, while the other subject reaches `suite:`.
  A refused pair add holds its project back with its own error and no `--keep-local` way forward, and the call ends without the held-back stop.
  A `NoOrch` row passes `noOrch` true to every `BeamUp` call.
  The pause re-run over a listed slug needs a persisted record, so `TestRunWorker` (card 7) carries it.
- `TestRunCheck` and `TestFailedEnd` drop their uses of `appendHolds` and `holdsFileName` with the holds file; every other assertion stays.
  `hubDepsAs` drops the `Holds` wiring.
- `TestAdoptIntegration_SecondCloneTakesOver` (tier1) expects the owner entry turned into one `adopt` entry and `adopt_paused`, and no `holds.json`.
  The first clone's adopt back now ends on the held-back stop, so the test reads its error, naming `--keep-local` and `--drop-local`, in place of failing on it.
  Its `PauseLifted` assertion goes with the field.

**ImpactSummary:** adoptcli's takeover is restructured into ordered steps and the hold files are gone; `hubDeps` is the one production wiring touched, and the envelope's per-slug `held` field becomes `adopt_paused`.
