# Card 7 — runner-engine-set

**Edit:**
- `internal/shuttleengine#Runner`
- `internal/shuttleengine#NewRunner` -> `func NewRunner(reed ReedOps, engines EngineSet, anchorPath, worktreeRoot string, cfg Config) *Runner`
- `internal/shuttleengine#NewDetachedRunner` -> `func NewDetachedRunner(reed ReedOps, engines EngineSet, anchorPath, worktreeRoot, paneCwd string, cfg Config) *Runner`
- `internal/shuttleengine#newFixture`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#TestNewRunner_RefusesUnusableToldPaths`
- `internal/shuttleengine#TestNewDetachedRunner_RefusesUnusableToldPaths`
- `internal/shuttleengine#TestNewDetachedRunner_AcceptsStandaloneShapeAndBothPaneCwdPositions`
- `internal/shuttleengine#TestRunner_Start_SweepSkipsEntirelyWithoutReedState`
- `internal/burlercli#burlerCLI.wireHub`
- `internal/burlercli#burlerCLI.wireStandalone`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#newParentReviewConfig` -> `func newParentReviewConfig(location *lyxcwd.Location, reedCfg reedengine.Config, cfg loomengine.Config, stencilsDir string, engines shuttleengine.EngineSet, shuttleCfg shuttleengine.Config) (parentreview.GateConfig, error)`
- `internal/loomcli#loomCLI.stopStepRuns`
- `internal/loomcli#TestNewParentReviewConfig_ReviewerFromResolver`
- `internal/loomcli#TestNewParentReviewConfig_KeepAliveInterval`
- `internal/loomcli#TestSmokeBurlerReview_AttachesToALiveReviewerInsteadOfRespawning`
- `internal/loomcli#TestSmokeLeadSeat_HarvestsAFinishedRunWithReedStateGone`
- `internal/loomcli#TestSmokeLeadSeat_RerunRemovesTheSupersededStrand`
- `internal/loomcli#TestSmokeGate_RepromptsThroughARealPaneAndFixesTheArtifact`
- `internal/loomcli#TestSmokeParentReview_NoticeReachesTheWriterPaneAndApproveLetsTheRunThrough`
- `internal/orchcli#orchCLI`
- `internal/orchcli#Command`
- `internal/orchcli#newLiveFixture`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenCLI.loadedTacklerDeps`
- `internal/tendercli#shuttlePair`
- `internal/tendercli#newShuttlePair`
- `internal/webstercli#websterCLI.wireHub`
- `internal/webstercli#websterCLI.wireStandalone`
- `internal/webstercli#newVerbsFixture`
- `internal/webstercli#newResetFixture`
- `internal/shuttlecli#shuttleCLI`
- `internal/shuttlecli#Command`
- `internal/shuttlecli#newFakeClockRunner`
- `internal/shuttlecli#TestRunCmd_EffortAndFocusFlags`
- `internal/shuttlecli#newSmokeRunner`
- `internal/websterengine_test#newRecoverFixtureOver`
- `internal/burlerengine_test#newClusterSmokeEngine`
- `internal/burlerengine_test#TestSmokeBurlerRoundToyFixture`
- `internal/seatengine_test#TestSmokeSeatsChairAsksAdvisor`
- `internal/treadleengine_test#TestSmokeJudgeCirclingToyFixture`
- `internal/shuttleengine/claudeengine_test#replayCorpusCase`
- `internal/shuttleengine/claudeengine#TestStart_LoneEOLMarkBoxTakesStartupSends`

**Uses:**
- `plan:internal/shuttleengines#New`
- `plan:internal/testkit/shuttlefake#EngineSet`

**Intent:** One `Runner` over the engine set: `NewRunner` and `NewDetachedRunner` take an `EngineSet` instead of an `Engine` (D4), and every production construction site builds its set through the registry (D1).

`Runner` gains `engines EngineSet`.
In this card every engine consult inside the package still reads one engine, the set's default, which both constructors resolve once with `engines.Lookup("")` into the existing `engine` field; card 8 replaces that field with per-run resolution.
A default that does not resolve is a construction verdict like a told-path one: it is joined into `toldErr`, so every public entry point refuses with the lookup's error and the constructors stay total.
That join is a fail-loud wiring guard with no per-row test: `shuttleengines.New` refuses a `cfg.Engine` naming no registered engine (card 5's `build`), so no production input reaches it, and card 1's `TestEngineSet_Lookup` already pins the lookup's error.
The constructors' doc comments name the set.

Every production site that called `claudeengine.NewFromConfig(shuttleCfg)` for a runner calls `shuttleengines.New(shuttleCfg)` and returns its error the way that wiring returns a config error:
- `burlerCLI.wireHub` and `wireStandalone`; `battenCLI.wire` and `loadedTacklerDeps`; `loomCLI.stopStepRuns`.
- `loomCLI.wire` passes the set to the runner and to `newParentReviewConfig`, and hands `RunDeps.Engine` the set's default engine until card 11.
  `newParentReviewConfig` resolves the writer's prompt-cache lifetime from the set's default engine, the writer spec naming none; a lookup error takes the existing lifetime-error branch, a Warn with keep-alive off, a wiring guard no production input reaches for the reason above.
- `orchCLI` gains `engines shuttleengine.EngineSet`, which `Command`'s `PersistentPreRunE` fills and builds the runner over; card 12 reads it.
- `newShuttlePair` builds the runner over `engines`, and `shuttlePair` drops its `engine` field.
- `websterCLI.wireHub` and `wireStandalone` build the runner over the set and pass `setRunner` the set's default engine until card 11.
- `shuttleCLI` drops its `engine` field, and `Command` builds the runner over `c.engines`.
After this card no production file outside `internal/shuttleengines` and the engine's own directory imports `claudeengine` but `orchcli/resumecontext.go`, which card 12 moves.

Test callers: `newFixture` keeps its `engine Engine` parameter and builds the runner over `NewEngineSet("fake", map[string]Engine{"fake": engine})`, so the package's tests are unchanged;
the four constructor tests in `internal/shuttleengine` wrap their engine the same way.
`internal/shuttleengine/doc.go`'s engine paragraph says the runner takes an `EngineSet`, which `internal/shuttleengines` builds from `shuttle.yaml`'s `engine:` key.
Every caller in another package wraps the engine it passes with `shuttlefake.EngineSet`, the tmux-tier smoke tests included, and `TestNewParentReviewConfig_*` and the parent-review smoke test pass `shuttlefake.EngineSet` of the engine they passed before.

**ImpactSummary:** Re-signs the two Runner constructors and one loomcli helper; every caller in burlercli, loomcli, orchcli, battencli, tendercli, webstercli and shuttlecli, and every test caller across nine packages, follows mechanically with no behavior change on a single-engine set.
