# Card 8 — fixer-attempts-tracked

**Create:**
- `plan:internal/landingshed#TestFixerAttemptsLiveInTheRecordsDir` -> `func TestFixerAttemptsLiveInTheRecordsDir(t *testing.T)`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#FixerAttemptsPath`
- `internal/landingshed#Finalize.returningFromFixer`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#Finalize.gateAndFix`
- `internal/landingshed#Finalize.rebaseAfterParentMerge`
- `internal/landingshed#Finalize.runFixer`
- `internal/landingshed#Finalize.judgeFixerChange`
- `internal/landingshed#Finalize.haltReviewCapped`
- `internal/landingshed#Finalize.endSpent`
- `internal/landingshed#ParentAlign.removeAttempts`
- `internal/landingshed#ParentAlign.readAttempts`
- `internal/landingshed#ParentAlign.align`
- `internal/landingshed#ParentAlign.judgeFixerChange`
- `internal/landingshed#fixerRun.run`
- `internal/landingshed#AcceptFixReview`
- `internal/landingshed#newFixerRunFixture`
- `internal/landingshed#newMergeInFixture`
- `internal/landingshed#newTestDeps`
- `internal/testkit/envkit#LandingDeps`
- `internal/loomcli#landingDeps`
- `internal/loomcli#circlingState.preRun`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`

**Uses:**
- `internal/landingshed/doc.go`
- `internal/loomcli/doc.go`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#TestStatusCommitPathspec`

**ImpactSummary:** the fixer attempt records move from loom's scratch directory to loom's durable directory, which every status commit already stages.

**Intent:**
Move the fixer attempt counts of `internal/landingshed/fixerattempts.go` into the pair's records, committed with the transition (D3).

`Deps` gains `RecordsDir`, the told absolute directory of the run's tracked landing records.
Every read, write and removal of a fixer attempt record builds its path with `FixerAttemptsPath(deps.RecordsDir, row)` in place of `deps.ScratchDir`;
`ScratchDir` keeps only the conflict-resolution report, which is a cache.
`FixerAttemptsPath`'s doc names its first argument the records directory.
`AcceptFixReview` takes the records directory, and `circlingState.preRun` passes `loomengine.LoomDurableDir`.
`landingDeps` sets `RecordsDir` to `loomengine.LoomDurableDir(l)`, the `_lym/loom` directory `statusCommitPathspec` already stages whenever it holds a file.
So each record is committed with the run's next transition and reaches another machine with the pair's records.
The test fixtures that build a `Deps` set `RecordsDir` beside `ScratchDir`.

Bound: a landing already in flight when this lands reads no count it left in the scratch directory, so its current row entry gets the cap once more.

`TestFixerAttemptsLiveInTheRecordsDir` (tier 0) runs a fixer turn over the fixture and asserts the record sits under `RecordsDir` and nothing under `ScratchDir`.

**Commit:** 8: the fixer attempt counts are tracked in loom's durable directory
