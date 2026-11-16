# Card 13 — fixer-attempts-tracked

**Create:**
- `plan:internal/shedrun#FixerAttemptsDir` -> `func FixerAttemptsDir(l *lymcwd.Location, runID string) string`
- `plan:internal/shedrun#FixerAttemptsRel` -> `func FixerAttemptsRel(l *lymcwd.Location, runID string) string`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#FixerAttemptsPath`
- `internal/landingshed#AcceptFixReview`
- `internal/landingshed#Finalize.endSpent`
- `internal/landingshed#Finalize.gateAndFix`
- `internal/landingshed#Finalize.haltReviewCapped`
- `internal/landingshed#Finalize.judgeFixerChange`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#Finalize.readAttempts`
- `internal/landingshed#Finalize.rebaseAfterParentMerge`
- `internal/landingshed#Finalize.returningFromFixer`
- `internal/landingshed#Finalize.runFixer`
- `internal/landingshed#ParentAlign.align`
- `internal/landingshed#ParentAlign.judgeFixerChange`
- `internal/landingshed#ParentAlign.readAttempts`
- `internal/landingshed#ParentAlign.removeAttempts`
- `internal/landingshed#fixerRun.run`
- `internal/landingshed#fixerSession.chargeTurn`
- `internal/landingshed#fixerSession.judge`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#newFinalizeGateFixture`
- `internal/landingshed#newFixerRunFixture`
- `internal/landingshed#newMergeInFixture`
- `internal/landingshed#TestAcceptFixReview`
- `internal/loomcli#landingDeps`
- `internal/loomcli#circlingState.preRun`
- `internal/loomcli#statusCommitPathspec`
- `internal/testkit/envkit#LandingDeps`
- `internal/shedrun#TestPathConstructors`

**Extends:**
- `internal/shedrun/doc.go`
- `internal/landingshed/doc.go`

**Uses:**
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/loomcli/doc.go`

**Intent:** The landing rows' fixer attempt counts become tracked records in the pair's records, committed with the transition (D3), so a resume on another machine never grants a fresh cap.

`FixerAttemptsDir` is `fixer-attempts` under `RunDir`, and `FixerAttemptsRel` its anchor-relative pathspec.
`landingshed.Deps` gains `AttemptsDir`, the told tracked directory of the attempt records; `ScratchDir` keeps the fixer reports and the fix-review files.
Every landingshed member listed here builds its attempts path from `AttemptsDir` instead of `ScratchDir`;
`FixerAttemptsPath` and `AcceptFixReview` name their first parameter the attempts directory, and their doc comments say so.
`landingDeps` fills `AttemptsDir` with `shedrun.FixerAttemptsDir(l, shedrun.SelfRunID)`,
and its `CommitStatus` closure commits `statusCommitPathspec(l, shedrun.SelfRunID)` instead of the status file alone, so the counts written before a merge are committed with it.
`statusCommitPathspec` adds `FixerAttemptsRel` when the directory holds a file or the records HEAD tracks one under it, so a removal commits too.
`circlingState.preRun` hands `AcceptFixReview` the attempts directory.
The test fixtures and `envkit.LandingDeps` set `AttemptsDir` to a temporary directory beside their scratch directory.

Tests, tier0: `TestPathConstructors` pins the directory and its rel;
`TestAcceptFixReview` writes and reads the record under the attempts directory and asserts the scratch directory holds no attempts file;
the existing fixer and Finalize tests keep passing through their fixtures.
The two-machine read is the same records commit card 12's integration test covers for the records branch.

**ImpactSummary:** Moves the attempt records from the loom scratch directory to the run's tracked records across the landing rows' attempt reads and writes; caps and their judgement are unchanged.

**Commit:** 13: fixer-attempts-tracked
