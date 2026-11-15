# Card 16 — fixer-attempts-tracked

**Create:**
- `internal/landingshed#moveScratchAttempts`

**Edit:**
- `internal/landingshed#Deps`
- `internal/landingshed#FixerAttempts`
- `internal/landingshed#FixerAttemptsPath`
- `internal/landingshed#ParentAlign.removeAttempts`
- `internal/landingshed#ParentAlign.readAttempts`
- `internal/landingshed#ParentAlign.align`
- `internal/landingshed#ParentAlign.judgeFixerChange`
- `internal/landingshed#Finalize.returningFromFixer`
- `internal/landingshed#Finalize.pushParent`
- `internal/landingshed#Finalize.gateAndFix`
- `internal/landingshed#Finalize.rebaseAfterParentMerge`
- `internal/landingshed#Finalize.readAttempts`
- `internal/landingshed#Finalize.runFixer`
- `internal/landingshed#Finalize.judgeFixerChange`
- `internal/landingshed#Finalize.haltReviewCapped`
- `internal/landingshed#Finalize.endSpent`
- `internal/landingshed#fixerRun.run`
- `internal/landingshed#AcceptFixReview`
- `internal/landingshed#newTestDeps`
- `internal/landingshed#NewTestDeps`
- `internal/landingshed#finalizeGateFixture.seed`
- `internal/landingshed#finalizeGateFixture.attempts`
- `internal/landingshed#TestFinalize_GateAndFixer`
- `internal/landingshed#TestFixerRun_Outcomes`
- `internal/landingshed#mergeInFixture.seedAttempts`
- `internal/landingshed#mergeInFixture.attemptsOnDisk`
- `internal/landingshed#TestMergeIn_Fixer`
- `internal/loomcli#landingDeps`
- `internal/loomcli#circlingState.preRun`

**Uses:**
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestLandingDeps_EveryFieldPopulated`
- `internal/loomcli#TestWire_BeamRecipe`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_SamplerRecipe`
- `internal/testkit/envkit#TestFullEnv_FillsEverySeam`
- `internal/testkit/envkit#TestNilSeams_ReportsExactPaths`
- `internal/testkit/envkit#TestNilSeams_SkipsNilLegalPaths`
- `internal/loomcli/doc.go`
- `internal/landingshed/doc.go`

**Intent:** The fixer attempt counts move from the run's scratch into the pair's records, committed with the transition (D3).
- `Deps` gains `RecordsDir`, the told tracked directory the run's transition commit stages; `ScratchDir` keeps the conflict-resolution and fixer reports.
- Every listed member builds the attempts path from `RecordsDir` where it uses `ScratchDir` today; `FixerAttemptsPath`'s parameter and doc, and `FixerAttempts`' file note, say records, not scratch.
- `moveScratchAttempts` moves a row's record found only under the scratch directory into the records directory, once; `ParentAlign.readAttempts` and `Finalize.readAttempts` call it first, so a run in flight across the change keeps its spent count and no resume grants a fresh cap.
- `loomcli`'s `landingDeps` sets `RecordsDir` to `loomengine.LoomDurableDir(l)`, which `statusCommitPathspec` already stages on every transition; `circlingState.preRun` passes the same directory to `AcceptFixReview`.
- `newTestDeps` and `NewTestDeps` set `RecordsDir` to a temporary directory apart from the scratch directory, and the listed fixtures and tests read and seed the attempts under `RecordsDir`, so a member left on `ScratchDir` fails them.

Tests, tier0:
- `TestMergeIn_Fixer` and `TestFinalize_GateAndFixer` gain a row each that seeds an attempts record under the scratch directory only, as a run in flight across the change has it, calls the row, and finds the record under `RecordsDir` with its spent count kept and none left in scratch.
- The existing rows of those tests and of `TestFixerRun_Outcomes` read the attempts from `RecordsDir`.

**ImpactSummary:** One Deps field and the attempts path's directory change across landingshed's fixer members; loomcli fills the field, the circling accept follows, and the fixer tests read the new directory.
