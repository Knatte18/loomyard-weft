# Card 8 — per-run-engine

**Edit:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#OpenSpec`
- `internal/shuttleengine#OpenResult`
- `internal/shuttleengine#Runner`
- `internal/shuttleengine#NewRunner`
- `internal/shuttleengine#NewDetachedRunner`
- `internal/shuttleengine#Run`
- `internal/shuttleengine#Runner.start`
- `internal/shuttleengine#Runner.checkResume` -> `func (r *Runner) checkResume(spec Spec, engine Engine) (string, error)`
- `internal/shuttleengine#Runner.skillLoader` -> `func (r *Runner) skillLoader(engine Engine) (SkillLoader, error)`
- `internal/shuttleengine#Runner.sessionCycler` -> `func (r *Runner) sessionCycler(engine Engine) (SessionCycler, error)`
- `internal/shuttleengine#Runner.newSendContext` -> `func (r *Runner) newSendContext(state RunState, runDir string) (sendContext, error)`
- `internal/shuttleengine#Runner.reconstruct` -> `func (r *Runner) reconstruct(candidate attachCandidate, normalized Spec, gate GateSpec) (*Run, error)`
- `internal/shuttleengine#Runner.attach`
- `internal/shuttleengine#Runner.Open`
- `internal/shuttleengine#Runner.Interrupt`
- `internal/shuttleengine#Runner.InterruptSettle`
- `internal/shuttleengine#Runner.PaneRunningTurn`
- `internal/shuttleengine#Runner.Send`
- `internal/shuttleengine#Runner.SendConditioned`
- `internal/shuttleengine#Runner.Inject`
- `internal/shuttleengine#Runner.LoadSkills`
- `internal/shuttleengine#Runner.ClassifySkillLoad`
- `internal/shuttleengine#Runner.ReadEvents`
- `internal/shuttleengine#Runner.SessionState`
- `internal/shuttleengine#Runner.ContextTokens`
- `internal/shuttleengine#Runner.CompactedSince`
- `internal/shuttleengine#Runner.SessionIdle`
- `internal/shuttleengine#Runner.sendSessionSequence`
- `internal/shuttleengine#Runner.TypeColor`
- `internal/shuttleengine#Run.newSendContext`
- `internal/shuttleengine#Run.loadSkillsThenPrompt`
- `internal/shuttleengine#Run.DeliverPendingPrompt`
- `internal/shuttleengine#Run.loadSkillTurn`
- `internal/shuttleengine#Run.Interrupt`
- `internal/shuttleengine#Run.watchDialog`
- `internal/shuttleengine#Run.logSessionState`
- `internal/shuttleengine#Run.boundaryIdle`
- `internal/shuttleengine#Run.startHoldReleased`
- `internal/shuttleengine#Run.pollEventsTick`
- `internal/shuttleengine#Run.armHoldGuard`
- `internal/shuttleengine#Run.checkLivenessTick`
- `internal/shuttleengine#Run.sessionGone`
- `internal/shuttleengine#Run.continueAPIErrorStall`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#fixtureSettings`
- `internal/shuttleengine#newFixture`
- `internal/shuttleengine#fixture.newRun`
- `internal/shuttleengine/doc.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/shuttleengine#Runner.engineFor` -> `func (r *Runner) engineFor(rs RunState) (Engine, string, error)`
- `plan:internal/shuttleengine#withEngines` -> `func withEngines(engines EngineSet) fixtureOpt`
- `internal/shuttleengine/enginedispatch_test.go`

**Intent:** Pick each run's engine by name: `Start` and `Open` resolve the spec's engine and record it, and every other engine-consulting operation resolves the engine its run's record names (D4, D5).

Fields:
- `Spec` and `OpenSpec` gain `Engine string`: the registered engine name the run uses, empty for the set's default; `validate` does not inspect it.
- `OpenResult` gains `Engine`, the resolved name: `Open` writes no run record, so its result and its `opened session` log line are where it records the engine.
- `Runner` drops the interim `engine` field card 7 added, and the constructors keep joining an unresolvable default into `toldErr`.
- `Run` gains `engine Engine` and `engineName string`, the engine the run is driven through and its resolved name.

`engineFor(rs)` resolves `r.engines.Lookup(rs.Engine)` and returns the engine with its resolved name, the default's name for a record with no `engine` key.

Start and Open, which act before any record exists:
- `start` resolves `spec.Engine` through the set right after `spec.validate`, before the skills check, the resume check, the pause wait, the run directory and any pane, so an unknown name is refused with the lookup's error, naming the registered engines, and creates nothing.
  It checks the skill loader and the resume against that engine, prepares the launch through it, writes the resolved name into `RunState.Engine`, logs it on `run started`, and sets the run's `engine` and `engineName`.
- `Open` resolves `spec.Engine` the same way before the pause check's side effects and the strand, and asserts `SessionOpener` on that engine.

Operations on a recorded run resolve the engine from the record through `engineFor`, and refuse with the lookup's error when the record names an unregistered engine:
- `reconstruct` resolves the candidate's engine and returns its error; `attach` returns that error from each of its four reconstruct sites.
  When the caller's normalized spec names an engine resolving to a different name than the record's, `reconstruct` logs a Warn naming both and the run directory, and the record's engine wins, because the live session belongs to it; a respawn after a failed attach is a new `Start`, which uses the spec's engine.
- `Runner.newSendContext` resolves the state's engine and returns the error, and each of its callers returns it; `Run.newSendContext` takes the run's own engine.
- `Interrupt`, `PaneRunningTurn`, `Inject`, `ReadEvents` and `SessionState` resolve from the record `FindRun` already reads; `Inject` moves its interrupt-key refusal after that read, since the key it refuses is the recorded engine's.
- `LoadSkills` resolves the record's engine before asking it for the skill loader.
- `SessionIdle` resolves the record's engine when `FindRun` finds one and the set's default when it does not, the strand-with-no-record case that today reads the pane probe.
- `sendSessionSequence` and `TypeColor` resolve through `newSendContext`.
- Every `run.runner.engine` read on a `Run` (`loadSkillsThenPrompt`, `DeliverPendingPrompt`, `loadSkillTurn`, `Interrupt`, `watchDialog`, `logSessionState`, `boundaryIdle`, `startHoldReleased`, `pollEventsTick`, `armHoldGuard`, `checkLivenessTick`, `sessionGone`, `continueAPIErrorStall`, `finalize`) reads `run.engine`.
- `InterruptSettle`, `ContextTokens`, `CompactedSince` and `ClassifySkillLoad`, which have no guid yet, resolve the set's default until card 9 gives them one.
- Stop, `StopStrand`, the step-stop sweep and strand removal consult no engine and resolve none, so a record naming an unregistered engine never blocks a stop or a teardown.

Tests: `fixtureSettings` gains `engines`, set by `withEngines`, and `newFixture` builds its runner over it when set, else over its one engine as card 7 does; `fixture.newRun` sets the run's `engine` and `engineName` from the runner's default.
`enginedispatch_test.go` declares `dispatchEngine`, a test engine embedding `*fakeEngine` that records, per engine, every seam method a call consults, and that answers with values distinct per instance: its own interrupt key from `InterruptSequence`, its own `InterruptSettle`, and the `SessionCycler` (`IdleSession`, `ContextTokens`, `CompactedSince`), `SkillLoader`, `SessionSignalParser` and `SessionOpener` methods, each recording its call.
A row asserts on the per-engine call record, so a call that consults the wrong engine shows there whatever the shared reed fake captured.
It adds `TestRunner_EngineSetDispatch`, table-driven over a set of two `dispatchEngine`s named `a` (default) and `b`:
- `Start` with `Spec.Engine` `a` and `b` each prepares through that engine only and records its name in `run.json`; an empty `Spec.Engine` uses `a`;
- `Open` with `b` and with no engine returns `OpenResult.Engine` `b` and `a`, and `Open` naming a registered engine without `SessionOpener` (a plain `fakeEngine` registered as `p`) is refused with no `AddStrand` call;
- `Start` and `Open` naming `c` fail with `ErrEngineNotRegistered`, naming `a, b`, with no `AddStrand` call;
- after a `Start` on `b`: `Runner.Send`, `Runner.SendConditioned`, `Runner.Interrupt`, `Runner.Inject` (its key list holding `a`'s interrupt key passes, `b`'s is refused), `Runner.PaneRunningTurn`, `Runner.ReadEvents`, `Runner.SessionState`, `Runner.SessionIdle`, `Runner.LoadSkills`, `Runner.TypeColor`, `Runner.ClearSession`, `Run.DeliverPendingPrompt` and an `Attach` of the same output files each consult `b` and never `a`;
- `Runner.SessionIdle` on a live strand with no run record consults `a`;
- an `Attach` by a fresh runner whose spec names `a` over a running record naming `b` waits through `b` and logs one Warn naming both engines;
- a record with no `engine` key is driven through `a`;
- a record naming `c`: each engine-consulting method of the after-`Start` row and an `Attach` over that record fail with `ErrEngineNotRegistered` and send no key, while `StopStrand` and `StopStepRuns` succeed and remove its strand, the record seeded with the `StepTie` the sweep is told so the sweep reaches it.

`internal/shuttleengine/doc.go`'s engine paragraph says `Start` and `Open` resolve `Spec.Engine`/`OpenSpec.Engine`, empty meaning the default, and record the name in `run.json`'s `engine` (in `OpenResult` for `Open`), that every other engine-consulting operation resolves the engine its run's record names, an absent key reading as the default and an unregistered name being refused, and that stop and teardown resolve none.
Raise `internal/shuttleengine` in `test-budget.yaml` from 175 to 176.

**ImpactSummary:** Internal to shuttleengine: five unexported helpers re-signed with their in-package callers, three exported structs gain an Engine field, the package doc gains the per-run engine, and no caller outside the package changes.
