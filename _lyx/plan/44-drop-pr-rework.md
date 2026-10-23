# Card 44 — drop-pr-rework

**Delete:**
- `internal/loomshed#NewReworkPlanGate`
- `internal/loomshed#reworkFixture.readRecord`
- `internal/loomshed#TestNewReworkPlanGate`
- `internal/loomshed#reworkFixture.producer`
- `internal/loomshed#reworkFixture.session`
- `internal/loomshed#reworkInner`
- `internal/loomshed#reworkInner.Call`
- `internal/loomshed#reworkFixture.call`
- `internal/loomshed#reworkFixture.mustDone`
- `internal/loomshed#TestPRRework_FirstRoundMovesGenerationZero`
- `internal/loomshed#TestPRRework_TwoRoundsKeepOneGenerationEach`
- `internal/loomshed#TestPRRework_CrashResumeConverges`
- `internal/loomshed#TestPRRework_SupersedingRejectionKeepsTheRound`
- `internal/loomshed#TestPRRework_RecordlessLowerRoundIsNotResumed`
- `internal/loomshed#TestPRRework_TornRecordIsAnError`
- `internal/loomshed#TestPRRework_ClasslessRecordAtHeadIsNotCommitted`
- `internal/loomshed#TestPRRework_ClassRecorded`
- `internal/loomshed#TestPRRework_CollisionNamesBothAndWayForward`
- `internal/loomshed#TestPRRework_BusyWebsterRefusalReachesTheOperator`
- `internal/loomshed#TestPRRework_ReentryAfterCommit`
- `internal/loomshed#TestPRRework_SecondRejectionAtSameHeadRuns`
- `internal/loomshed#TestPRRework_AbsentRecordNoRound`
- `internal/loomshed#TestPRRework_AbsentRecordWithCommittedRound`
- `internal/loomshed#TestPRRework_MalformedRecordStuck`
- `internal/loomshed#TestPRRework_NilSeamIsNamedError`
- `internal/loomshed#TestLatestRejectReview`
- `internal/loomengine/rework.go`
- `internal/loomengine/rework_test.go`
- `internal/loomengine#LoomRejectionPath`
- `internal/landingshed/rejection.go`
- `internal/landingshed/rejection_test.go`
- `internal/loomcli#pendingRejectionReader`
- `internal/loomcli#rejectionClearer`
- `internal/loomcli#rejectionPending`
- `internal/loomcli#TestGateParity_ReworkPlanGate`
- `contracts/stencils/loom/loom-template-rework.md`
- `contracts/stencils#LoomTemplateRework`

**Edit:**
- `internal/loomshed/prrework.go`
- `internal/loomshed#reworkFixture`
- `internal/loomshed#newReworkFixture`
- `internal/loomshed#reworkFixture.deps`
- `internal/loomcli#pendingRejectionSetup`
- `internal/loomcli#TestValidatePlanCmd_ReworkReportsGateFindingsForFirstCardMismatch`
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#retiredLoomKeys`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestMigrateConfig_RetiredKeys`
- `internal/loomengine#TestProducerSpecs_SkillsAndParentDirective`
- `internal/loomengine#TestResolveReviewAndJudge_MalformedSpec`
- `internal/loomengine#TestLandingAndApprovalAccessors`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#resolvePlainGateClosure`
- `internal/shedrecipe#TestResolveGateSpec_OtherNamesCarryNoFinal`
- `internal/shedrecipe#TestResolveGateSpec_PlanGatesRequirePlanIndex`
- `internal/shedrecipe#TestResolveGateSpec_UnknownNameListsSixValues`
- `internal/shedrecipe#TestResolveGateSpec_ValidOneEntryPerVocabularyName`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#TestCommitStatusSeam_Real`
- `internal/loomcli#TestWire_Real_ReworkRoundsArchiveGenerations`
- `internal/loomcli#TestWire_Real_PlanReviewSkipFollowsGenerationClass`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#ReworkDeps`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`

**Extends:**
- `internal/loomshed/doc.go`

**Intent:** PR-Rework goes with its `rework-plan` gate, its stencil and its `rework` key (D13), now that no reject reaches it.
In `prrework.go` the producer (`prRework`), its constructor and every method, its compile-time assertion, the pending rejection, the told-values type `ReworkTold`, and the helpers only the producer read (`rejectionIdentity`, `LatestRejectReview`, `latestPriorChild`, `readRoundRecord`, `writeRoundRecord`, `holdsEntry`, `move`) go, while the round-record reading the generation code uses stays; `PRReworkDeps` keeps only the seams the Plan-Review skip seam and the plan gates still read (`PlanDir`, `ReworkDir`, `ReviewsDir`, `ReadCommitted` and what `PlanReviewSkippable` needs), so the rework rounds a run already recorded still classify, and `lyx loom validate-plan --rework` still validates through `ValidateReworkPlan`.
The `rework-plan` gate entry goes from `resolvePlainGateClosure`, so a row naming it is the entry's unknown-name error, and `NewReworkPlanGate` goes; `Env.ReworkTable` goes and `wire` stops building it and its rework-alias resolution.
`loomengine`'s rework table, its model resolution and the `rework` key go: `Config` drops `Rework`, the template drops the key, and `retiredLoomKeys` gains `rework`, so the migration drops it from every file, profile and run_settings document.
The rejection record, its path accessor and its loomcli readers go with their last reader; `statusCommitPathspec` no longer holds the rework sweep back on a pending rejection.
The `loom-template-rework` stencil, its registration and its `rework` role key go.
The rework fixture keeps what the generation tests use, and the PR-Rework tests go.

`doc.go` drops PR-Rework.

**ImpactSummary:** One producer, one gate entry, one stencil and one loom.yaml key go, with the rejection record; the rework round reading that Plan-Review's skip seam and the `--rework` validation use stays.
