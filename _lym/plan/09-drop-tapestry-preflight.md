# Card 9 — drop-tapestry-preflight

**Create:**
- `internal/preflightshed#NewPreflightWithBatcher`
- `internal/preflightshed#TestPreflight_BatcherCheck`

**Edit:**
- `internal/preflightshed#NewPreflight`
- `internal/preflightshed/doc.go`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#preflightEntry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#simpleEntryCases`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#TestFinalizeEntry_LandingRejected`
- `internal/shedrecipe/doc.go`
- `internal/testkit/envkit#nilLegal`
- `internal/loomshed#InterruptPolicies`
- `internal/loomshed#NewWebsterProducer`
- `internal/loomshed#websterProducer.Call`
- `internal/loomshed#TestCancellation_RealProducersReturnErrorNotStuck`
- `internal/loomshed/doc.go`
- `internal/loomengine/status.go`
- `internal/loomengine#TestVerifySeedOwnership`
- `internal/loomrecipe#RetiredRows`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#TestRecipe_SeedAndResumeRowNamesExist`
- `internal/loomrecipe#TestRecipe_StructuralCheckHasNoFindings`
- `internal/loomrecipe#TestNew_ConstructionFailureNamesOffendingRow`
- `internal/loomrecipe#TestResume_DoesNotRestartAtRowOne`
- `internal/loomrecipe#TestBounceRouting_EmptyTargetBlocksInstead`
- `internal/loomrecipe#TestResume_CrashRecoveryRecallsUnconditionally`
- `internal/loomrecipe#TestSequence_FullRunBlocksAtMergeIn`
- `internal/loomrecipe#TestRecipeKeys`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#awaitRunLock`
- `internal/loomcli#TestRenameRetiredRows`
- `internal/loomcli#TestObserveEntry_GotoIsNotACrashResume`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#waitForCurrentProducer`
- `internal/shedrun/doc.go`
- `internal/hubreconcile/doc.go`
- `contracts/specs/shed-recipe-spec.md`
- `internal/reportengine#loomPhases`
- `internal/reportengine#TestComputeTiming`
- `internal/reportengine#loomRowPhases`
- `contracts/recipes/tapestry-recipe.yaml`
- `contracts/specs/loom-status-spec.md`
- `docs/overview.md`
- `internal/reportengine/doc.go`
- `internal/loomengine#VerifySeedOwnership`
- `internal/loomrecipe#New`
- `internal/loomcli#lockFree`
- `pattern/PATTERN-cwd-resolution.md`
- `internal/loomrecipe/doc.go`
- `internal/loomrecipe#TestEntryNumbers`
- `internal/battencli#TestOverviewRunCells`
- `internal/battencli#TestDobbyRunCells_ReadsTheRunsFromDisk`
- `internal/battencli#TestDobbyRunCells_ResolvesTheChildPairAgainInEachRunShedSpell`

**Delete:**
- `internal/loomshed/tapestrypreflight.go`
- `internal/loomshed/tapestrypreflight_test.go`
- `internal/loomshed#NameTapestryPreflight`
- `internal/loomshed#TestTapestryPreflight_StuckSurfacesItsFailures`
- `internal/shedrecipe#tapestryPreflightEntry`
- `internal/shedrecipe#TestTapestryPreflightEntry_RelativeAnchorPath`
- `internal/loomengine#CheckSeed`
- `internal/loomengine/coherence.go`
- `internal/loomengine/coherence_test.go`
- `internal/loomengine/seed_test.go`
- `internal/loomengine/report.go`

**Uses:**
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestFullEnv_PathFieldsAbsoluteUnderOneRoot`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/testkit/envkit#TestSeamMarkers_ReflectiveTestsAreMarked`
- `internal/loomrecipe#HasEngine`

**Intent:** The Tapestry-Preflight row and its producer go, with its seed check, and its `batcher.yaml` load moves into Preflight for a recipe holding an `Implementer` row (D10).

Preflight:
- `NewPreflightWithBatcher` builds the Preflight producer with an optional `batcher` check; `NewPreflight` keeps its signature and calls it with nil.
- The check runs after the hub reconcile and before `preflight.Check`, so it is the early failure a broken `batcher.yaml` gave, before any LLM spends tokens. A nil seam runs no check. An error maps to Stuck, empty Path, reason `active batchifier did not resolve: <error>` followed by `; way forward: fix batcher.yaml (its active: key or the profile it names), then re-step`, or, for an error wrapping `batcher.ErrRetiredKey`, `, then re-step` after the error's own reconcile way forward, the two texts Tapestry-Preflight gave; a cancelled context wins as on every other exit.
- `shedrecipe.Env` gains `PreflightBatcher func() error`, "the batcher.yaml load Preflight runs, set by the wiring for a recipe with an Implementer row, nil otherwise"; `preflightEntry` passes it to `NewPreflightWithBatcher`. It is listed in `envkit`'s `nilLegal`, since a darn run leaves it nil.
- `wire` sets it to `func() error { _, err := batcher.Active(dirs); return err }` when `loomrecipe.HasEngine(c.recipe, "Implementer")` (empty recipe read as tapestry), and leaves it nil otherwise, so darn and pawl skip the check.

Removal:
- The `TapestryPreflight` registry entry and `tapestryPreflightEntry` go; `Env.StatusPath`/`StatusLockPath` stay (loomrecipe's coherence check reads them) and their field docs drop TapestryPreflight; `Env.ConfigDirs`' doc names Webster alone.
- `tapestry-recipe.yaml`'s Preflight row's `on_done` is Discussion-Write and the Tapestry-Preflight row goes, with the header's lines about it.
- `loomengine.CheckSeed`, `checkCoherence` (coherence.go), the four loom check IDs and the aliases of `report.go`, and their tests go; `status.go`'s package doc stops naming the seed check, and `VerifySeedOwnership` stays.
- `NameTapestryPreflight` goes from the name constants, `InterruptPolicies` and `loomPhases`.
- `RetiredRows("tapestry")` maps `Tapestry-Preflight` and `Loom-Preflight` onto `Preflight`, so a run parked on Tapestry-Preflight resumes at Preflight and runs the batcher load again.

Bound: with the half-finished checks gone, a run whose current row is Preflight but whose status holds history past Preflight, a set `start_sha` or `pause_requested` proceeds into the row after Preflight; such a status reaches Preflight only on a fresh seed, a resume of a run blocked at Preflight or parked on Tapestry-Preflight, `lym loom goto --to Preflight`, or a status file edited outside those routes (the decision record's D10 bound).

Tests:
- `TestPreflight_BatcherCheck` (tier0) drives a producer over a temporary non-repository cwd: a failing seam is Stuck with the batchifier reason and its way forward; an error wrapping `batcher.ErrRetiredKey` ends with `, then re-step`; a nil seam and a passing one never yield that reason.
- `TestResume_DoesNotRestartAtRowOne` and `TestBounceRouting_EmptyTargetBlocksInstead` drop their `shed1.Producers[1]` substitution, which stood in for Tapestry-Preflight and would now replace Discussion-Write. The D10 bound, a Preflight-current status with history past Preflight proceeding, needs no row of its own: after this card no row of the recipe reads that history, which `wantProducerTable` pins by holding no Tapestry-Preflight row.
- `TestWire_DefaultConfig` asserts `env.PreflightBatcher` is set on a tapestry seed, and `TestWire_DarnRecipe` that it is nil on a darn seed, so a broken `batcher.yaml` fails Preflight on tapestry and not on darn.
- `TestPreflight_BatcherCheck` also holds a row whose seam cancels the context and then returns an error, so the call passes the entry check and reaches the cancel check after the batcher seam; it asserts the returned error is the cancellation error and the outcome is not Stuck. The test also asserts the failing row logs the row's name and the batcher error text, as Tapestry-Preflight logged it; it captures the process-global logger, so it does not call `t.Parallel`.
- `TestRenameRetiredRows` gains a row renaming a status parked on `Tapestry-Preflight` to `Preflight`; `TestRecipeKeys`' tapestry `RetiredRows` row adds it.
- The table and literal fixtures that name the row or its producer drop it (`wantProducerTable`, the shedrecipe entry tables and `TestNames`, the loomrecipe tests listed, `TestCancellation_RealProducersReturnErrorNotStuck`, `TestComputeTiming`, `TestObserveEntry_GotoIsNotACrashResume`, `TestResume_CrashRecoveryRecallsUnconditionally`'s doc, `TestSequence_FullRunBlocksAtMergeIn`'s fixture, rewritten onto Preflight then Discussion-Write, and `TestVerifySeedOwnership`'s comment); `awaitRunLock`'s, `waitForCurrentProducer`'s and the webster producer's comments drop the row. `internal/shedrun/doc.go` stops naming `CheckSeed`; `VerifySeedOwnership`'s doc drops its `CheckSeed` and Tapestry-Preflight comparisons and keeps what the function answers; `New`'s doc in `internal/loomrecipe` drops `tapestryPreflightEntry` and the Tapestry-Preflight hazard, stating the status-path check as keeping `env`'s copy equal to Shed's paths; `internal/loomrecipe/doc.go`'s retired-row sentence maps `Loom-Preflight` and `Tapestry-Preflight` onto `Preflight`; `lockFree`'s comment drops the row, and so does the file comment of `TestTapestryPreflight_StuckSurfacesItsFailures`' file, which that deletion leaves; `internal/reportengine/doc.go`'s tapestry setup sentence drops Tapestry-Preflight; `pattern/PATTERN-cwd-resolution.md` lists two tiers, `lymcwd.Resolve` then `preflight.Check`, and says an orchestrator needs tier 2; `internal/hubreconcile/doc.go` names Preflight's batcher load as the retired-key backstop, and `shed-recipe-spec.md` drops the row. `tools/tapestryrename` keeps the old names, since it records a rename already made.
- With Tapestry-Preflight gone every tapestry entry number after Preflight drops by one, so `TestEntryNumbers`' tapestry map renumbers (Discussion-Write 2 through Friction-Reflect 15), and the battencli stage cells built from those numbers follow: `TestOverviewRunCells`, `TestDobbyRunCells_ReadsTheRunsFromDisk` and `TestDobbyRunCells_ResolvesTheChildPairAgainInEachRunShedSpell` expect `6.5 Plan-Write` in place of `6.6 Plan-Write`, and `TestOverviewRunCells` expects `6.6 Plan-Burler-Review` in place of `6.7 Plan-Burler-Review`.
`loom-status-spec.md` drops the Tapestry-Preflight row and CheckSeed's verdicts; `docs/overview.md`'s tapestry row list drops it.

**ImpactSummary:** One row, its producer and the seed check go; Preflight gains an optional batcher seam the wiring fills for Implementer recipes, and fixtures naming the row lose it.
