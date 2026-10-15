# Card 1 — Spec embeds ModelChoice, Result carries the engine

**Edit:**
- `internal/shuttleengine#Spec`
- `internal/shuttleengine#OpenSpec`
- `internal/shuttleengine#ModelChoice`
- `internal/shuttleengine#Result`
- `internal/shuttleengine#Run.finalize`
- `internal/shuttleengine#Run.identity`
- `internal/shuttleengine/doc.go`
- `internal/shuttleengine#TestRunner_EngineSetDispatch`
- `internal/shuttleengine#dispatchRig.start`
- `internal/shuttleengine#TestRunner_Requires`
- `internal/shuttleengine#requiresRig.startRecord`
- `internal/shuttleengine/claudeengine#TestPrepare_RejectedBeforeArtifacts`
- `internal/shuttleengine/claudeengine#TestPrepare_ThreadsSpecIntoLaunchCmds`
- `internal/shuttleengine/claudeengine#TestPrepare_ResumeSessionID`
- `internal/shuttleengine/claudeengine#TestOpenLaunch`
- `internal/sheddriver#tacklerSpec`
- `internal/treadleengine#runJudgeCall`
- `internal/treadleengine#runTargeting`
- `internal/websterengine#recoverSpawn`
- `internal/websterengine#Run`
- `internal/alignment#Spec`
- `internal/loomcli#arbiterSpec`
- `internal/loomcli#driverSpec`
- `internal/orchcli#orchCLI.orchSpec`
- `internal/frictionengine#buildReflectionSpec`
- `internal/frictionengine#TestReflect_OneNote_SpawnsAndArchives`
- `internal/mergeresolve#buildConflictSpec`
- `internal/mergeresolve#TestBuildConflictSpec_SkillsAndParentDirective`
- `internal/shuttleengine#TestNewModelChoice`
- `internal/seatengine#seatSpec`
- `internal/shuttlecli#shuttleCLI.runCmd`
- `internal/shuttlecli#shuttleCLI.openCmd`
- `internal/shedadapters#Bouncer.runSeedSpawn`
- `internal/shedadapters#Bouncer.judgeCall`
- `internal/shuttlecli#TestSmokeGuardrailQuestionHoldsRunUntilAnswered`
- `internal/shuttlecli#startMidTurnCountingRun`
- `internal/shuttlecli#TestSmokePasteSubmitAndCollapse`
- `internal/shuttlecli#TestSmokeTurnsLongBackgroundShell`
- `internal/shuttlecli#TestSmokeTurnsShellFinishesMidTurn`
- `internal/shuttlecli#TestSmokeTurnsMonitor`
- `internal/shuttlecli#TestSmokeTurnsFork`
- `internal/shuttlecli#TestSmokeTurnsForkPermissionAsk`
- `internal/shuttlecli#TestSmokeTurnsMessageMidTurn`
- `internal/shuttlecli#TestSmokeTurnsSkillLoad`
- `internal/shuttlecli#TestSmokeTurnsCompact`
- `internal/shuttlecli#TestSmokeTurnsSessionHooks`
- `internal/shuttlecli#TestSmokeTurnsHookCapture`
- `cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/shuttlecli#TestOpenCmd_ModelAliasEngine` -> `func TestOpenCmd_ModelAliasEngine(t *testing.T)`

**Intent:**
D1 and the shuttle half of D4.
`Spec` and `OpenSpec` drop their own `Model`, `Effort`, `Version` and `Engine` fields and embed `ModelChoice`, so reads such as `spec.Model` and `spec.Engine` keep working through promotion and a construction sets `ModelChoice: choice`.
The field docs move from `Spec` onto `ModelChoice`'s fields: `Engine` is the registered engine name, empty for the engine set's default, which `Start` and `Open` refuse when the set does not hold it and `validate` never inspects; `Model`, `Effort` and `Version` keep their provider-vocabulary docs (the engine is the sole validator).
No provider name enters the type (`PATTERN-shuttle-provider-seam`).

`Result` gains `Engine`: the resolved name of the engine the run was driven through, `run.engineName`, which is the record's `engine` value and the set's default name for a record naming none.
`Run.finalize` and `Run.identity` set it, so every terminal result and every mechanism-failure identity carries it; a `Start` refused before a run exists returns the zero `Result`, whose `Engine` is empty.
`doc.go` states the embedding and `Result.Engine`.

Every `Spec` and `OpenSpec` literal that set one of the four fields is rewritten to set `ModelChoice` (a promoted field cannot be named in a composite literal), each production site keeping the values it passes today:
`tacklerSpec`, `runJudgeCall`, `runTargeting`, `alignment.Spec`, `arbiterSpec`, `driverSpec`, `orchSpec`, `seatSpec`, the Bouncer's seed and judge specs and shuttle's `run` verb build `shuttleengine.ModelChoice{Model: ..., Effort: ..., Version: ...}` from the fields their carriers hold now (later cards hand them a whole choice).
The sites that resolve a `modelspec.Resolved` themselves build their choice with `shuttleengine.NewModelChoice(resolved)`, so they now carry the alias's engine: `webster`'s Master spec in `Run` and recovery spec in `recoverSpawn`, `buildReflectionSpec`, `buildConflictSpec`, and `lyx shuttle open`, whose `openCmd` sets `spec.ModelChoice = shuttleengine.NewModelChoice(resolved)` in place of its three-field assignment.
The test literals listed are rewritten the same way with unchanged values, the tmux-tier shuttlecli smoke specs keeping `smokeClaudeModel` inside the choice so `TestSmokeSuite_EveryRealClaudeSpawnPinsTheModel` still finds the constant at every site.

Tests:
- `TestRunner_EngineSetDispatch` gains a subtest: a run started with `Engine: "b"` and driven to done returns `Result.Engine == "b"`, one started with an empty engine returns `"a"` (the set's default name), and a run whose `Wait` ends on a mechanism failure returns its identity with the engine set.
  A further row rewrites a started run's `run.json` with no `engine` key, as a record written before `engine-registry`, attaches to it with `Runner.Attach` and drives it to done: `Result.Engine` is `"a"`, the default's name.
- `TestNewModelChoice` gains an end-to-end row: `modelspec.Parse("claude:some-model[high]")` resolved through an empty `modelspec.Registry` and mapped by `NewModelChoice` gives `Engine` `claude`, model `some-model` and effort `high`.
  It is the escape-form row for every resolver at once, since card 13's scan holds every resolver to `Resolve` then `NewModelChoice`.
- `TestReflect_OneNote_SpawnsAndArchives` asserts the reflection spec's `Engine` is the friction alias's engine (`claude` for the test registry's alias), and `TestBuildConflictSpec_SkillsAndParentDirective` asserts the conflict spec's `Engine` is the conflict alias's engine and its `Requires` is empty (merge-resolve declares nothing, D5).
- `TestOpenCmd_ModelAliasEngine` builds a `shuttleCLI` over a real `Runner` on `shuttlefake.Reed` and an engine set whose default is `fake` and which also registers a second `shuttlefake.Engine` under `claude`, with `configDirs` on an empty temp anchor so the registry is the built-in set.
  `lyx shuttle open --role explore --model opus` must fail with the open refusal naming engine `"claude"` (neither fake has a session opener), which proves the alias's engine reached `Open`; without `--model` it names `"fake"`.
  `cmd/lyx/testdata/test-budget.yaml`'s `internal/shuttlecli` row rises by one for it.

**ImpactSummary:** shuttleengine's spawn value types change shape; every literal that set a model field across shuttle's callers is rewritten with unchanged values, reads are untouched by promotion, and `Result` gains one field.

**Commit:** 1: Spec and OpenSpec embed ModelChoice; Result carries the run's engine
