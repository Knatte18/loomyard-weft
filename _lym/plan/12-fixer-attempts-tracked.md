# Card 12 — fixer-attempts-tracked

**Create:**
- `plan:internal/shedrun#FixerAttemptsDir` -> `func FixerAttemptsDir(l *lymcwd.Location, runID string) string`
- `plan:internal/shedrun#FixerAttemptsRel` -> `func FixerAttemptsRel(l *lymcwd.Location, runID string) string`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#fixerRun.run`
- `internal/landingshed#NewFinalize`
- `internal/landingshed#Finalize.runFixer`
- `internal/landingshed#NewParentAlign`
- `internal/landingshed#ParentAlign.removeAttempts`
- `internal/landingshed#ParentAlign.readAttempts`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#newFixerRunFixture`
- `internal/landingshed#newMergeInFixture`
- `internal/landingshed#newFinalizeGateFixture`
- `internal/loomcli#landingDeps`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#circlingState.preRun`
- `plan:internal/loomcli#TestPullRequestRecord_ReachesAnotherCloneThroughOrigin`
- `internal/battencli#battenCLI.wire`
- `internal/battencli#battenRunCommitPaths`
- `internal/battencli#TestNewCommitStatusSeam`
- `internal/shedrun#TestPathConstructors`

**Extends:**
- `internal/shedrun/doc.go`
- `internal/landingshed/doc.go`

**Uses:**
- `internal/loomcli/doc.go`
- `internal/landingshed#FixerAttemptsPath`
- `internal/landingshed#AcceptFixReview`
- `internal/landingshed#TestFixerAttempts`
- `internal/landingshed#TestFixerRun_Outcomes`
- `internal/landingshed#TestMergeIn_Fixer`
- `internal/landingshed#TestFinalize_GateAndFixer`
- `internal/landingshed#TestAcceptFixReview`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`

**Intent:**
The fixer attempt counts of `internal/landingshed/fixerattempts.go` move into the pair's records and are committed with the transition (D3), so a resume on another machine never grants a fresh cap.
`shedrun.FixerAttemptsDir` is `fixer-attempts` under `shedrun.RunDir`, and `FixerAttemptsRel` its anchor-relative form.
`landingshed.Deps` gains `AttemptsDir`, the tracked directory the attempt records live in; every `FixerAttemptsPath` call in `fixerRun.run`, `Finalize` and `ParentAlign` takes it in place of `ScratchDir`, and `NewFinalize` and `NewParentAlign` refuse an empty one as they refuse a missing seam.
`loomcli.landingDeps` sets it to `shedrun.FixerAttemptsDir(l, shedrun.SelfRunID)`, `battenCLI.wire` to `shedrun.FixerAttemptsDir(location, slug)`, and `circlingState.preRun` passes it to `AcceptFixReview` in place of the loom scratch directory.
`statusCommitPathspec` and `battenRunCommitPaths` stage `FixerAttemptsRel` when the directory holds a file (`shedrun.HoldsFile`), so each transition's commit carries the counts.
The record format is unchanged; a record left in the scratch directory by an older build is not read, which grants that one landing entry a fresh cap once.
The test fixtures set `AttemptsDir`; `TestNewCommitStatusSeam` (tier0) pins the new path in batten's commit list; `TestPathConstructors` pins the new constructors;
`TestPullRequestRecord_ReachesAnotherCloneThroughOrigin` (tier1) also writes an attempt record and reads it on the other clone after a pull.

**ImpactSummary:** fixer attempt records move from the scratch directory to the run's tracked records and ride every transition commit.

**Commit:** 12: Fixer attempt counts live in the run's records and ride its transition commits
