# Card 16 — parent-align-row

**Rename:**
- `//internal/landingshed/mergein.go` -> `//internal/landingshed/parentalign.go`
- `//internal/landingshed/mergein_test.go` -> `//internal/landingshed/parentalign_test.go`
- `//internal/landingshed/mergein_fixer_test.go` -> `//internal/landingshed/parentalign_fixer_test.go`

**Create:**
- `plan:internal/landingshed#ParentAlign` -> `type ParentAlign struct`
- `plan:internal/landingshed#NewParentAlign` -> `func NewParentAlign(deps Deps) (*ParentAlign, error)`
- `plan:internal/landingshed#parentAlignName` -> `const parentAlignName = "Parent-Align"`
- `plan:internal/landingshed#retiredMergeInName` -> `const retiredMergeInName = "Merge-In"`
- `plan:internal/landingshed#FixerModeParentAlign` -> `const FixerModeParentAlign FixerMode = "parent-align"`
- `plan:internal/loomshed#NameParentAlign` -> `const NameParentAlign = "Parent-Align"`
- `plan:internal/shedrecipe#parentAlignEntry` -> `func parentAlignEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `plan:internal/gateslot#renameGateKeys` -> `func renameGateKeys(gates *yaml.Node) []string`
- `plan:internal/loomrecipe#LoomRetiredRows` -> `func LoomRetiredRows() map[string]string`
- `plan:internal/loomcli#loomCLI.renameRetiredRows` -> `func (c *loomCLI) renameRetiredRows(renames map[string]string) error`
- `plan:internal/loomcli#TestRenameRetiredRows` -> `func TestRenameRetiredRows(t *testing.T)`
- `plan:internal/reportengine#retiredMergeInRow` -> `const retiredMergeInRow = "Merge-In"`

**Delete:**
- `internal/loomshed#NameMergeIn`
- `internal/landingshed#FixerModeMergeIn`
- `internal/shedrecipe#mergeInEntry`
- `internal/loomcli#loomCLI.renameRetiredDarnRows`

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `contracts/stencils/landing/landing-template-fixer.md`
- `internal/loomshed#InterruptPolicies`
- `internal/landingshed#FixerTable`
- `internal/landingshed#HandsBack`
- `internal/landingshed#TestHandsBack`
- `internal/landingshed#TestFixerTable`
- `internal/loomshed#TestDarnWrite_Call`
- `internal/loomrecipe#TestEntryNumbers`
- `internal/testkit/envkit#LandingDeps`
- `internal/testkit/envkit#fixedGateLists`
- `internal/loomcli#recipeRow`
- `internal/loomcli#TestPeersVerb`
- `internal/landingshed#fixerRunFixture.execute`
- `internal/landingshed#callProducer`
- `internal/landingshed#TestPreMerge_ClearsOwnParkedMergeIn`
- `internal/landingshed_test#TestMergeIn_ConflictSessionLoadsLanguageSkills`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#simpleEntryCases`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#TestSimpleEntries_HappyPath`
- `internal/gateslot#retiredGateLists`
- `internal/gateslot#MigrateConfig`
- `internal/gateslot#goReadGates`
- `internal/gateslot/template.yaml`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#TestMigrateConfig`
- `internal/loomcli#loomCLI.armAt`
- `internal/loomcli#fixerReportInput`
- `internal/loomcli#isLandingProducer`
- `internal/loomcli#TestReviewInputSeams`
- `internal/reportengine#retiredRowNames`
- `internal/reportengine#recipePhaseTables`
- `internal/reportengine#darnPhases`
- `internal/reportengine#RunRow.classifyBatten`
- `internal/reportengine#TestComputeTiming`
- `internal/reportengine#loomRowPhases`
- `internal/shedadapters#AmendmentNote`
- `internal/fabricengine#BranchOutcome.stopWayForward`
- `internal/fabricengine#busyWayForward`
- `internal/fabricengine_test#liveWayForward`
- `internal/battencli#TestChildParentLock`
- `internal/loomcli#newArbiterFixture`
- `internal/loomcli#TestArbiterVerb_SpawnPath`
- `internal/loomcli#TestArbiterVerb_VerdictOutcomes`
- `internal/loomcli#TestFinalizeKeepsLock`
- `internal/loomcli#TestLatestDarnOutcome`
- `internal/loomcli#TestLoomRerouteCapWayForward`
- `internal/loomcli#TestRecipeRouting`
- `internal/loomcli#TestResumeVerb`
- `internal/loomcli#TestRunDriverSpawnAndWait_MidMerge_CleanPairProceeds`
- `internal/loomcli#TestRunDriverSpawnAndWait_MidMerge_ProbeErrorRefuses`
- `internal/loomcli#TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals`
- `internal/loomrecipe#deriveSequenceOrder`
- `internal/loomrecipe#TestCoverageGuard_MergeInAndFinalizeRowNamesMatchTheirProducerIdentity`
- `internal/loomrecipe#TestEscalation_PerchBudgetOneWithAutoAccept`
- `internal/loomrecipe#TestIsFinalizeRow`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestNew_MergeInAndFinalizeAreRealProducers`
- `internal/loomrecipe#TestNew_MissingLandingClosureReturnsError`
- `internal/loomrecipe#TestResume_DoesNotRestartAtRowOne`
- `internal/loomrecipe#TestResume_PauseStopsAtBoundaryAndClearsFlag`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#TestSequence_FullRunBlocksAtMergeIn`
- `internal/loomrecipe#TestZeroBudget_BouncerSettlesAtOnce`
- `internal/loomrecipe#wantProducerTable`
- `internal/reportengine#darnRowPhases`

**Intent:** D11: loom's `Merge-In` row becomes `Parent-Align`, with everything exclusive to that row, and in-flight runs are carried across.
fabric's own merge-in (`lyx fabric merge-in`, `fabricengine.MergeIn`, `MergeInAt`, `MergeInOnly`, `MergeInProgress`, `MergeVerbMergeIn`) and the prose "merge-in" for any git merge of the parent keep their names.
The hub has no `rename_tools`, so every rename below is a hand edit.

- **Recipe.** In `loom-recipe.yaml` the row is `Parent-Align`, engine `ParentAlign`, between Webster and Code-Bouncer; Webster's `on_done` and Finalize's `on_reroute` name it, and every comment naming the row follows.
  The header's restart rule says a run parked on `Merge-In` is carried across by rename, not restarted, through `LoomRetiredRows`; row positions are unchanged, so `lyx loom peers` reports the same positions.
- **Producer.** `git mv` the three landingshed files first, then retarget identifiers only: `MergeIn` to `ParentAlign`, `NewMergeIn` to `NewParentAlign` (its refusals name it), `mergeInName` to `parentAlignName = "Parent-Align"`, the fixture and test identifiers that name the type, and the file's comments that name the row.
  `FixerModeMergeIn` becomes `FixerModeParentAlign`, `"parent-align"`, which `FixerTable` switches on and `landing-template-fixer.md` names (`In \`parent-align\` mode`); `loomshed.NameMergeIn` becomes `NameParentAlign`.
  `HandsBack` matches `parentAlignName` and `retiredMergeInName` alike, since batten and arbiterstop read a child's status file directly, before any loom verb has renamed a run halted on `Merge-In` at upgrade; `TestHandsBack` keeps its `Merge-In` rows wanting true and gains the same rows for `Parent-Align`.
  `envkit`'s `LandingDeps` gate key and the `fixedGateLists` entry are `parent-align`, so a recipe built over the kit resolves the row's key; `TestEntryNumbers` keys entry 8 `Parent-Align`; `TestDarnWrite_Call`'s `wantReasonLacks` names `Parent-Align`.
  The landing row's gate key is its lower-cased row name, so the row now runs `parent-align`, and the merge fixer runs the key of the row that spawned it.
- **Attempt record.** The row's attempt record is `FixerAttemptsPath(scratch, parentAlignName)`.
  `ParentAlign.readAttempts` reads `FixerAttemptsPath(scratch, retiredMergeInName)` when its own is absent, and every site that removes its own record (the success path and the align path) also removes the old one, so a run halted spent or over-cap on `Merge-In` resumes on `Parent-Align` with its spent attempts, `Baseline`, `AlignFrom` and `Halt`; the entry number matches because the rename carries the row's history entries across.
  The decode refusal names and tells the operator to remove the file it read, the old one when that is the record it fell back to.
  `TestMergeIn_Fixer` gains rows over `mergeInFixture`: a carried record under the old name only is read with every field, the halt it holds still halts, an undecodable old record is refused naming the old path, and the old file is gone after the success path and after the align path each remove the row's record.
- **Registry.** `shedrecipe`'s engine key `MergeIn` becomes `ParentAlign`, built by `parentAlignEntry` in place of `mergeInEntry`; `minimalEntries`, `simpleEntryCases` (the default gate key is now `parent-align`), `TestNames` and `TestSimpleEntries_HappyPath` follow.
- **Gate key.** `gate.yaml`'s template key `merge-in` becomes `parent-align`, in the template's `gates`, its comment and `goReadGates`, and the retired `landing` list migrates to `parent-align`.
  `renameGateKeys`, called from `MigrateConfig` after the list migration, renames a hub's `gates.merge-in` to `gates.parent-align` with its value and the line `gates.merge-in: renamed to gates.parent-align`, and drops the old key beside an existing new one; `TestMigrateConfig` gains both rows and `templateDefaults` the new key, so a hub `gate.yaml` holding `merge-in` is renamed by reconcile and its tiers take effect.
- **In-flight runs.** `LoomRetiredRows` returns `{"Merge-In": NameParentAlign}`.
  `renameRetiredRows(renames)` is `renameRetiredDarnRows` taking its table, and `armAt` calls it with `DarnRetiredRows()` for a darn run and `LoomRetiredRows()` for a loom run, so a status file whose `current_producer`, `entered_row`, pause fields or history name `Merge-In` resumes on `Parent-Align`; a live driver holding the run lock is left alone with a warning, as today.
  `TestRenameRetiredRows` writes a loom status parked on `Merge-In` and asserts every field reads `Parent-Align` after the call, and that a darn status still renames `Darn`.
  `lyx loom peers` reads a peer's status unrenamed, so `recipeRow` maps a producer through `LoomRetiredRows` and `DarnRetiredRows` before its lookup, and a peer parked on `Merge-In` is listed at Parent-Align's position; `TestPeersVerb` gains that peer.
- **Code-Perch input.** `fixerReportInput` reads `Parent-Align-fixer-report.md`, and the old `Merge-In-fixer-report.md` when the new one is absent, so a run past the row at upgrade still gives the Code-Perch its latest fixer report; `TestReviewInputSeams` gains the old-name row and a row holding both reports, where the new one wins.
- **Reports.** `retiredMergeInRow` joins `retiredRowNames` in the merge phase, `recipePhaseTables` keeps the `Merge` family beside the `Parent` one, and `classifyBatten`'s verify rows hold `NameParentAlign` and `retiredMergeInRow`, so archived `Merge-In` rows are filed in the merge phase and among the verify rows, as `retiredPublishRow` is; `TestComputeTiming` and `loomRowPhases` gain an archived `Merge-In` row beside the new one.
- **Strings naming the row.** `AmendmentNote`'s sentence and fabricengine's two way-forward lines name `Parent-Align`; `liveWayForward` follows.
- The remaining test members that name `loomshed.NameMergeIn`, `landingshed.MergeIn` or `NewMergeIn` take the new names.

**ImpactSummary:** One recipe row, engine key, producer, fixer mode, gate key and row constant renamed across landingshed, loomshed, shedrecipe, gateslot, loomrecipe, loomcli and reportengine, with read-time fallbacks for the old attempt record, fixer report and status rows.
