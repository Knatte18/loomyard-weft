# Card 2 — constructor-row-identity

**Create:**
- `plan:internal/shedrecipe#RowIdentity` -> `type RowIdentity struct`
- `plan:internal/shedrecipe#testReviewSegment` -> `const testReviewSegment = "Test-Review"`
- `plan:internal/shedrecipe#reviewRow` -> `func reviewRow(name string) RowIdentity`

**Edit:**
- `internal/shedrecipe#Constructor` -> `type Constructor func(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#preflightEntry` -> `func preflightEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#publishEntry` -> `func publishEntry(_ RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#mergeInEntry` -> `func mergeInEntry(_ RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#prGateEntry` -> `func prGateEntry(_ RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#prAlignEntry` -> `func prAlignEntry(_ RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#finalizeEntry` -> `func finalizeEntry(_ RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#frictionReflectEntry` -> `func frictionReflectEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#loomPreflightEntry` -> `func loomPreflightEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#batchifierEntry` -> `func batchifierEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#stubEntry` -> `func stubEntry(row RowIdentity, cfg Config, _ Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#websterEntry` -> `func websterEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#prReworkEntry` -> `func prReworkEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#darnWriteEntry` -> `func darnWriteEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#describeEntry` -> `func describeEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#discussionWriteEntry` -> `func discussionWriteEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#planWriteEntry` -> `func planWriteEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#multiLLMEntry` -> `func multiLLMEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#bouncerEntry` -> `func bouncerEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#burlerReviewEntry` -> `func burlerReviewEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#burlerFixEntry` -> `func burlerFixEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#worktreeCreateEntry` -> `func worktreeCreateEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#innerRunEntry` -> `func innerRunEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#seedChildEntry` -> `func seedChildEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#worktreeTeardownEntry` -> `func worktreeTeardownEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#landEntry` -> `func landEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#openWindowEntry` -> `func openWindowEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#seedIDEEntry` -> `func seedIDEEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#openIDEEntry` -> `func openIDEEntry(row RowIdentity, cfg Config, env Env) (shedengine.ShedProducer, error)`
- `internal/shedrecipe#registry`
- `internal/shedbuild#Build`
- `internal/loomcli#newPRFixture`
- `internal/shedrecipe#TestEveryEntry_RejectsUnknownConfigKey`
- `internal/shedrecipe#lifecycleEntryCase`
- `internal/shedrecipe#lifecycleEntryCases`
- `internal/shedrecipe#TestLifecycleEntries_HappyPath`
- `internal/shedrecipe#TestLifecycleEntries_RejectsEmptySlug`
- `internal/shedrecipe#TestLifecycleEntries_RejectsBadScratchDir`
- `internal/shedrecipe#TestLifecycleEntries_RejectsNilSeams`
- `internal/shedrecipe#TestInnerRunEntry_DriverExitGraceKey`
- `internal/shedrecipe#TestInnerRunEntry_NilNowIsAccepted`
- `internal/shedrecipe#TestInnerRunEntry_NilSleepIsAccepted`
- `internal/shedrecipe#TestInnerRunEntry_NoticeProbeKey`
- `internal/shedrecipe#TestInnerRunEntry_NoticeQuietKey`
- `internal/shedrecipe#TestInnerRunEntry_PollConfigKeys`
- `internal/shedrecipe#TestBouncerEntry_ApproveSeam`
- `internal/shedrecipe#TestBouncerEntry_ArtifactPathsResolveUnderAnchorPath`
- `internal/shedrecipe#TestBouncerEntry_BlockingEscalatesAndColorSegment`
- `internal/shedrecipe#TestBouncerEntry_CarryOver`
- `internal/shedrecipe#TestBouncerEntry_CirclingReasonSlugAndBudget`
- `internal/shedrecipe#TestBouncerEntry_ClusterExcludes`
- `internal/shedrecipe#TestBouncerEntry_CommitSeam`
- `internal/shedrecipe#TestBouncerEntry_ConstructionFailures`
- `internal/shedrecipe#TestBouncerEntry_EnvJudgeFallback`
- `internal/shedrecipe#TestBouncerEntry_HappyPath`
- `internal/shedrecipe#TestBouncerEntry_ParentNamePassesToJudgePrompt`
- `internal/shedrecipe#TestBouncerEntry_ReportNameKeyRejected`
- `internal/shedrecipe#TestBouncerEntry_ReportNamePinning`
- `internal/shedrecipe#TestBouncerEntry_RunDirectory`
- `internal/shedrecipe#TestBouncerEntry_SkipSeam`
- `internal/shedrecipe#lookupEntry`
- `internal/shedrecipe#captureReview`
- `internal/shedrecipe#captureFix`
- `internal/shedrecipe#TestBurlerFixEntry_GateConfig`
- `internal/shedrecipe#TestBurlerHalfEntries_ColorSegment`
- `internal/shedrecipe#TestBurlerHalfEntries_ConstructionFailures`
- `internal/shedrecipe#TestBurlerHalfEntries_EnvFallback`
- `internal/shedrecipe#TestBurlerHalfEntries_FileSetErrorsNameTheirOwnMap`
- `internal/shedrecipe#TestBurlerHalfEntries_RunDirectory`
- `internal/shedrecipe#TestBurlerHalfEntries_StrictUnknownKeys`
- `internal/shedrecipe#TestBurlerReviewEntry_ClusterFan`
- `internal/shedrecipe#TestBurlerReviewEntry_RubricStencil`
- `internal/shedrecipe#TestDarnWriteEntry_ConstructionFailures`
- `internal/shedrecipe#TestDarnWriteEntry_HappyPath`
- `internal/shedrecipe#TestDescribeEntry_ConstructionFailures`
- `internal/shedrecipe#TestDescribeEntry_HappyPath`
- `internal/shedrecipe#TestDiscussionWriteEntry_CallDone`
- `internal/shedrecipe#TestDiscussionWriteEntry_ConstructionFailures`
- `internal/shedrecipe#TestDiscussionWriteEntry_GateConfig`
- `internal/shedrecipe#TestDiscussionWriteEntry_HappyPath`
- `internal/shedrecipe#TestDiscussionWriteEntry_ParentReviewBeginRound`
- `internal/shedrecipe#gateCapableEntries`
- `internal/shedrecipe#TestGateCapableEntries_RejectLeftoverOldKeys`
- `internal/shedrecipe#TestMultiLLMEntry_ComposedTable`
- `internal/shedrecipe#TestMultiLLMEntry_ConstructionFailures`
- `internal/shedrecipe#TestPlanWriteEntry_CallAppendsPriorPlanBlock`
- `internal/shedrecipe#TestPlanWriteEntry_CallDone`
- `internal/shedrecipe#TestPlanWriteEntry_ConstructionFailures`
- `internal/shedrecipe#TestPlanWriteEntry_HappyPath`
- `internal/shedrecipe#TestPRReworkEntry_Config`
- `internal/shedrecipe#TestPRReworkEntry_ConstructionFailures`
- `internal/shedrecipe#TestPRReworkEntry_EvaluatesReworkTableWithToldValue`
- `internal/shedrecipe#TestPRReworkEntry_HappyPath`
- `internal/shedrecipe#simpleEntryCase`
- `internal/shedrecipe#simpleEntryCases`
- `internal/shedrecipe#TestSimpleEntries_HappyPath`
- `internal/shedrecipe#TestSimpleEntries_UnderfilledEnv`
- `internal/shedrecipe#TestBatchifierEntry_RelativeAnchorPath`
- `internal/shedrecipe#TestFinalizeEntry_LandingRejected`
- `internal/shedrecipe#TestFrictionReflectEntry_NilClosureRejected`
- `internal/shedrecipe#TestPRGateEntry`
- `internal/shedrecipe#TestPublishEntry_LandingRejected`
- `internal/shedrecipe#TestWebsterEntry_GateKeys`
- `internal/shedrecipe#TestWebsterEntry_SeamFields`
- `internal/shedrecipe#TestSwitchedEntries_BuildOverTheirOwnSeamsAndRefuseANilOne`
- `internal/shedrecipe#Lookup`

**Intent:** D2: the registry's `Constructor` learns its row's segment.
`RowIdentity` is a small struct with two string fields, `Name`, the recipe row's `name`, and `Segment`, its `segment:` value (empty for a row outside any segment); its doc comment says so.
`Constructor` takes a `RowIdentity` in place of the bare name, and every registry entry takes the new parameter; this card is a pure re-sign, so every entry reads `row.Name` exactly where it read `name` before (entries that discarded the name keep discarding it) and no entry reads `row.Segment` yet — card 4 makes the Bouncer, BurlerReview and BurlerFix entries read it.
`switchedEntry` keeps its `name string` parameter and its three callers pass `row.Name`.
The `registry` map literal, `Lookup` and `Names` keep their text (`PATTERN-shed-recipe-registry`); `registry` and `Lookup` are listed because they reference the re-signed `Constructor` type.
`Constructor`'s doc comment is rewritten for the new parameter, and the `recipe.go` file header names `RowIdentity` among the declared types.

`shedbuild.Build` calls each constructor with `shedrecipe.RowIdentity{Name: row.Name, Segment: row.Segment}`; the `Segment` it already copies onto `ProducerDef` is the same value.
`newPRFixture` in `internal/loomcli/parentreview_integration_test.go` (integration tag) builds its Discussion-Write row the same way.

Test call sites change mechanically.
`testReviewSegment` and `reviewRow(name)` live in the package's test fixture: `reviewRow` returns `RowIdentity{Name: name, Segment: testReviewSegment}`, the identity every Bouncer, BurlerReview and BurlerFix test row is built with, so card 4's segment requirement needs only a fixture `Segments` entry.
Every direct call of a Bouncer, BurlerReview or BurlerFix entry in this package's tests passes `reviewRow("<name>")`, and `captureReview` and `captureFix` keep their `name string` parameter and call their entry with `reviewRow(name)`; every other entry's test call passes `RowIdentity{Name: "<name>"}`.
`TestEveryEntry_RejectsUnknownConfigKey` calls each entry with `reviewRow("row-name")`, which every entry accepts.
The table-driven tests that call an entry through a `Constructor` value (`tt.entry(...)`, `lookupEntry(t, ...)(...)`, the conformance and switched tests' `constructor(...)`) pass a `RowIdentity` the same way, and the table types and builders holding a `Constructor` field or map (`lifecycleEntryCase`, `simpleEntryCase`, `gateCapableEntries` and the like) are listed because they reference the re-signed type, though their text needs no change.
Each test member whose body holds such a call or reference is listed on this card.
No assertion changes: the existing entry tests, the registry tests and `TestBuild_EveryRegisteredEngineBuilds` cover the re-sign.

**ImpactSummary:** Every registry entry and its one production caller `shedbuild.Build` re-signed, plus one integration-test caller in loomcli and the shedrecipe tests' direct calls; behavior is unchanged.

**Commit:** 2: shedrecipe: hand each constructor its row's name and segment
