# Card 10 — retire-burler-round

**Delete:**
- `internal/shedadapters#BurlerRunner`
- `internal/shedadapters#BurlerProducer`
- `internal/shedadapters#BurlerDeps`
- `internal/shedadapters#NewBurlerProducer`
- `internal/shedadapters#BurlerProducer.Call`
- `internal/shedadapters#BurlerProducer.doneExit`
- `internal/shedadapters#BurlerProducer.probeLiveRound`
- `internal/shedadapters#BurlerProducer.stopLiveHalf`
- `internal/shedadapters#BurlerProducer.resumeLiveRound`
- `internal/shedadapters#BurlerProducer.liveRoundErrorExit`
- `internal/shedadapters#describeHalves`
- `internal/shedadapters#newRoundUsage`
- `internal/shedadapters#TestBurlerProducer_ResumesBothLiveHalvesInsteadOfRespawning`
- `internal/shedadapters#TestBurlerProducer_NoLiveHalfSpawnsAttemptOne`
- `internal/shedadapters#TestBurlerProducer_ResumedRoundThatEndedDeadRespawnsFromAttemptOne`
- `internal/shedadapters#TestBurlerProducer_LiveRoundErrorNeitherArchivesNorSpawns`
- `internal/shedadapters#TestBurlerProducer_PartiallyLiveRoundIsStoppedAndRespawned`
- `internal/shedadapters#TestNewBurlerProducer_Validation`
- `internal/shedadapters#TestBurlerProducer_RoundScan`
- `internal/shedadapters#TestBurlerProducer_Hydration`
- `internal/shedadapters#TestBurlerProducer_Call_DoneReturnsStuckNeverDone`
- `internal/shedadapters#TestBurlerProducer_Call_WritesRoundUsageRecord`
- `internal/shedadapters#TestBurlerProducer_Call_ProfileCarriesDerivedFields`
- `internal/shedadapters#TestBurlerProducer_Call_ClusterExclusions`
- `internal/shedadapters#TestBurlerProducer_Call_RunOptsCarriesRoundTokenAndNoteID`
- `internal/shedadapters#TestBurlerProducer_Call_DiedThenDoneSucceedsWithRetry`
- `internal/shedadapters#TestBurlerProducer_Call_FailedRunIsAnError`
- `internal/shedadapters#TestBurlerProducer_Call_CancelledBeforeAnAttempt`
- `internal/shedadapters#TestBurlerProducer_Call_CancelledDuringFailedRoundArchives`
- `internal/shedadapters#TestBurlerProducer_Call_CancelledDuringCompletedRoundLeavesArtifacts`
- `internal/shedadapters#TestBurlerProducer_Gate_FailedGateMapsToStuckWithEmptyPointer`
- `internal/shedadapters#TestBurlerProducer_Gate_ProbeLiveRoundPassesGateAndMapsFailedGateIdentically`
- `internal/shedadapters#TestBurlerProducer_Call_ArchiveOnExit`
- `internal/shedadapters#TestBurlerProducer_Call_PreAttemptArchiveFailureHonoursCancellation`
- `internal/shedadapters#roundWith`
- `internal/shedadapters#roundWithStops`
- `internal/shedadapters#burlerProducerSpec`
- `internal/shedadapters#burlerProducerOpt`
- `internal/shedadapters#withBurlerProfile`
- `internal/shedadapters#withBurlerRunOpts`
- `internal/shedadapters#withBurlerModels`
- `internal/shedadapters#withBurlerClock`
- `internal/shedadapters#newBurlerProducer`
- `internal/testkit/shedfake#BurlerRunner`
- `internal/testkit/shedfake#BurlerRunner.Run`
- `internal/testkit/shedfake#BurlerRunner.ProbeRound`
- `internal/testkit/shedfake#BurlerRunner.Resume`
- `internal/testkit/shedfake#TestBurlerRunner_LastEntryRepeats`
- `internal/loomcli#probingBurlerRunner`
- `internal/loomcli#probingBurlerRunner.Run`
- `internal/loomcli#probingBurlerRunner.ProbeRound`
- `internal/loomcli#probingBurlerRunner.Resume`
- `internal/loomcli#TestSmokeBurlerRound_AttachesToALiveRoundInsteadOfRespawning`
- `internal/loomshed#NameDiscussionBurler`
- `internal/loomshed#NamePlanBurler`
- `internal/loomshed#NameWebsterBurler`
- `internal/loomshed#NameDarnBurler`
- `internal/burlerengine#burlerRolePrefix`
- `internal/burlerengine#Result`
- `internal/burlerengine#Profile.roundTable`
- `internal/burlerengine#Engine.Run`
- `internal/burlerengine#notStartedResult`
- `internal/burlerengine#reviewReady`
- `internal/burlerengine#roundAcceptance`
- `internal/burlerengine#roundAcceptance.accept`
- `internal/burlerengine#roundAcceptance.accepted`
- `internal/burlerengine#roundResult`
- `internal/burlerengine#Engine.concludeRound`
- `internal/burlerengine#Engine.decideRound`
- `internal/burlerengine#LiveRound`
- `internal/burlerengine#Engine.probeTable`
- `internal/burlerengine#Engine.ProbeRound`
- `internal/burlerengine#Engine.Resume`
- `internal/burlerengine#roundFilePaths`
- `internal/burlerengine#roundFilesIn`
- `internal/burlerengine#roundPrompts`
- `internal/burlerengine#roundSeatValues`
- `internal/burlerengine#composePrompt`
- `internal/burlerengine#Profile.validate`
- `internal/burlerengine#TestEngine_Run_SpecConstruction`
- `internal/burlerengine#TestEngine_Run_ForkSubagentsSpecWiring`
- `internal/burlerengine#TestEngine_Run_ClusterAuditPolicy`
- `internal/burlerengine#TestEngine_Run_ShuttleOutcomes`
- `internal/burlerengine#TestEngine_Run_GateOutcomes`
- `internal/burlerengine#TestEngine_Run_ReviewGateRepairsUnparseableReview`
- `internal/burlerengine#TestEngine_Run_MarkerLifecycle`
- `internal/burlerengine#TestEngine_Run_RoundFailureRules`
- `internal/burlerengine#TestEngine_Run_MaterializesInstructionFiles`
- `internal/burlerengine#TestEngine_Run_PatternDirectiveReachesInstruction1`
- `internal/burlerengine#TestEngine_Run_MaterializeFailure`
- `internal/burlerengine#TestEngine_ProbeRound`
- `internal/burlerengine#TestEngine_Resume`
- `internal/burlerengine#reviewRole`
- `internal/burlerengine#fixRole`
- `internal/burlerengine#fakeHandle.waitForMarker`
- `internal/burlerengine#reviewOrchestratorMarkerValues`
- `internal/burlerengine#fixOrchestratorMarkerValues`
- `internal/burlerengine#instruction2MarkerValues`
- `internal/burlerengine#instruction3MarkerValues`
- `internal/burlerengine_test#readyMarkerPath`
- `contracts/stencils#BurlerTemplateReviewOrchestrator`
- `contracts/stencils#BurlerTemplateFixOrchestrator`
- `contracts/stencils#BurlerStep2Review`
- `contracts/stencils#BurlerStep3Fix`
- `contracts/stencils/burler/burler-template-review-orchestrator.md`
- `contracts/stencils/burler/burler-template-fix-orchestrator.md`
- `contracts/stencils/burler/burler-step-2-review.md`
- `contracts/stencils/burler/burler-step-3-fix.md`
- `internal/burlermarker/doc.go`
- `internal/burlermarker/marker.go`
- `internal/burlermarker/marker_test.go`
- `internal/burlerengine#testRoundFilePaths`
- `internal/burlerengine#seatPrompts`

**Create:**
- `plan:internal/burlerengine#chairPrompt` -> `func chairPrompt(t *testing.T, stencilsDir, parentName string, p *Profile, table seatengine.Table) string`

**Edit:**
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#RoundFocus`
- `internal/shedadapters#roundComplete`
- `internal/shedadapters#writeRoundPair`
- `internal/shedadapters/burlerreview_test.go`
- `internal/shedadapters/doc.go`
- `internal/burlerengine#Profile`
- `internal/burlerengine#composeReviewPrompt`
- `internal/burlerengine#halfScript`
- `internal/burlerengine#fakeShuttle`
- `internal/burlerengine#fakeShuttle.scriptFor`
- `internal/burlerengine#fakeShuttle.initLocked`
- `internal/burlerengine#fakeShuttle.StartGated`
- `internal/burlerengine#fakeHandle.Wait`
- `internal/burlerengine#doneRound`
- `internal/burlerengine#newEngineTestProfile`
- `internal/burlerengine#newEngineWith`
- `internal/burlerengine#newValidProfileFixture`
- `internal/burlerengine#TestProfile_Validate`
- `internal/burlerengine#TestProfileValidate_ClusterExclude`
- `internal/burlerengine#newComposableProfile`
- `internal/burlerengine#combinedPrompt` -> `func combinedPrompt(t *testing.T, stencilsDir string, p *Profile, table seatengine.Table, prompt halfPrompt) string`
- `internal/burlerengine#TestComposePrompt_FocusDirective`
- `internal/burlerengine#TestComposePrompt_MinimalProfile`
- `internal/burlerengine#TestComposePrompt_ParentDirective`
- `internal/burlerengine#TestComposePrompt_FixScope`
- `internal/burlerengine#TestComposePrompt_ToolUse`
- `internal/burlerengine#TestComposePrompt_PriorRounds`
- `internal/burlerengine#TestComposePrompt_ClusterRules`
- `internal/burlerengine#TestComposePrompt_FrictionDirective`
- `internal/burlerengine#instruction1MarkerValues`
- `internal/burlerengine#TestTemplate_FillsWithAllMarkers`
- `internal/burlerengine#TestTemplate_OptionalDirectives`
- `internal/burlerengine#TestComposePrompt_ReadsEditedStencilFromDisk`
- `internal/burlerengine_test#TestSmokeBurlerRoundToyFixture`
- `internal/burlerengine_test#TestSmokeBurlerClusterCleanFan`
- `internal/burlerengine_test#TestSmokeBurlerClusterRogueFork`
- `internal/burlerengine/doc.go`
- `contracts/stencils#entries`
- `contracts/stencils#roleOpeningStencils`
- `contracts/stencils#wordingClaims`
- `contracts/stencils/burler/burler-step-1-explore.md`
- `contracts/stencils/friction/friction-directive-review-fix.md`
- `contracts/stencils/pattern/pattern-directive-review-fix.md`
- `cmd/lyx#pathRules`
- `cmd/lyx/testdata/test-budget.yaml`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`

**Uses:**
- `internal/shedadapters#readAcceptance`

**Intent:**
D8 and the retirement half of D1 and D2: card 4 moved every production caller to the split rows and card 5 moved `lyx burler run` to the halves, so the one-row round producer, the two-seat engine round, the reviewer-fixer channel and the ready marker go, with the old row-name constants.
They go in one card, after every card that edits code naming them, because the old producer, its fake and the old engine round reference one another.

Shed adapter side: `BurlerProducer`, its `BurlerRunner` seam, `BurlerDeps` and their helpers go; the round helpers the two rows share (`roundReviewPath`, `roundFixerReportPath`, `roundComplete`, `ParseRoundReviewName`, `highestCompleteRound`, `hydrationPaths`, `describeHalf`) stay in `burler.go`, and `newRoundUsage` goes with its one caller, since the rows build each half's usage with `newHalfUsage`.
`burler_test.go` loses its `BurlerProducer` tests, card 4 having dropped its compile-time assertion over the old fake, keeping the round-file writers the new rows' tests and `multillm_test.go` use; `attachprobe_test.go` loses its `BurlerProducer` tests and the `LiveRound` builders, keeping its probed handle; `bouncer_fixture_test.go` loses the `BurlerProducer` builder; `shedfake.BurlerRunner` goes with its test.
`loomcli`'s `smoke_attachprobe_test.go` loses `probingBurlerRunner`, its three methods and `TestSmokeBurlerRound_AttachesToALiveRoundInsteadOfRespawning`, which card 4's `TestSmokeBurlerReview_AttachesToALiveReviewerInsteadOfRespawning` replaces.
Card 3's row tests carry every kept case of the deleted tests.
D3's complete-round check reads the acceptance record here, once no producer that writes none is left: `roundComplete` also requires the round's acceptance record, read through `readAcceptance`, so `highestCompleteRound` and `hydrationPaths` treat a review and fixer report without one as an orphan; `writeRoundPair` writes the record with the pair, so the row tests' complete rounds stay complete; and `burlerreview_test.go` gains a row where a review and fixer report without a record are archived and reviewed again.
`BouncerConfig.ClusterExcludes`'s doc names the review row its `OnStuck` names, `RoundFocus`'s doc and `focus.go`'s header name the review row as the focus file's reader, and `internal/shedadapters/doc.go` drops its `BurlerProducer` sections and its ready-marker sentence.
`loomshed`'s four `Name*Burler` constants go; card 4 left them unreferenced.

Engine side: `burlerengine` loses `Engine.Run`, `ProbeRound`, `Resume`, `LiveRound`, `Result`, the round table, the round handoff (`reviewReady`, `roundAcceptance`, `concludeRound`, `decideRound`, `roundResult`, `notStartedResult`), the round prompt composition and the `burler` role prefix.
`Profile.validate` goes; `validateReview` and `validateFix` are the two validators, and `TestProfileValidate_ClusterExclude` drives `validateReview`, and `TestProfile_Validate` drives `validateReview` without a marker row, keeping card 2's `validateFix` rows.
`Profile.ReadyMarkerPath` goes, and so does `internal/burlermarker`, its only deriver, with its row in `cmd/lyx`'s `pathRules` and in `test-budget.yaml`.
`composeReviewPrompt` stops passing a friction directive, since `burler-step-1-explore` loses its `friction_directive` marker: only the fixer writes a friction note, and its directive rides `burler-step-fix`.
The four round stencils go (`burler-template-review-orchestrator`, `burler-template-fix-orchestrator`, `burler-step-2-review`, `burler-step-3-fix`) with their embedded vars, registry entries, the `burler-advisor-1` and `burler-chair` entries of `roleOpeningStencils`, and their `wordingClaims`.
`burler-step-1-explore`'s banner names the reviewer as its only reader; `friction-directive-review-fix`'s banner names `burler-step-fix` as its call site; `pattern-directive-review-fix` binds the reviewer alone and drops its fixer part, since the fixer no longer receives the PATTERN overview.

Engine tests: the round tests go, since card 2's half tests cover their behavior.
`engine_test.go`'s fake drops the marker wait (`halfScript`, `fakeShuttle`, `scriptFor` keyed on the two half roles, `initLocked`, `StartGated`, `fakeHandle.Wait`, `doneRound`, `newEngineTestProfile`, `newEngineWith`).
`prompt_test.go`'s composition tests drive `composeReviewPrompt`, or `composeFixPrompt` for the fix-scope and friction cases, over paths from `reviewFilesIn` or `fixFilesIn` and the half's table (`reviewTable` or `fixTable`) built from the half prompt's seat values.
`testRoundFilePaths` and `seatPrompts` go with the round types they name; `chairPrompt` renders the one chair seat's prompt of a half's table told a parent name, and `combinedPrompt` joins that chair prompt with every instruction file of the half prompt; `template_test.go` keeps the explore step's markers without `friction_directive`.
The `llm` smoke tests change only where the two-seat round they pinned is gone: the toy round runs `Engine.Review` then `Engine.Fix` with the review's digest, and the two cluster smoke tests run `Engine.Review`.

Docs: `internal/burlerengine/doc.go` describes the package as two halves on one-seat tables, the acceptance digest, the empty-review skip and the fix gates, keeping its finding-class, fix-scope, cluster and verdict sections, and drops the seat channel, the ready marker and the round sections.
`docs/overview.md` drops `internal/burlermarker` from the tree, names the review and fix row producers in the `internal/shedadapters` line and the adapter list, and describes burler and its execution stack as a reviewer session then a fixer session, with no marker.
`refusal-spec.md`'s out-of-scope paragraph names a changed review and a failed acceptance-record write as Burler's own round errors, in place of the skipped handoff and the ready marker.
The `internal/shedadapters`, `internal/testkit/shedfake` and `internal/burlerengine` rows of `test-budget.yaml` fall to the packages' new counts.

**ImpactSummary:** removes dead API in `shedadapters`, `shedfake`, `loomshed` and `burlerengine` and the whole `burlermarker` package; nothing in production references a deleted symbol after cards 4 and 5; `roundComplete`'s one behavioral change, requiring the acceptance record, reaches only the two Burler row producers through `highestCompleteRound` and `hydrationPaths`.

**Commit:** 10: burler: retire the one-row round, the two-seat engine round and the ready marker
