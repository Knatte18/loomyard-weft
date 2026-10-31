# Lens review

Five lens forks ran: coverage, row-validity, forced-edits, guard-reach and way-forward; every fork returned a report.
Duplicates are merged under the first lens that raised them.

- coverage 1, MEDIUM — fixed as proposed: card 11's existing-pair row asserts `Commit` is still called once, so an uncommitted pair commits on resume.
- coverage 2, MEDIUM — fixed differently: proposed a row walking the whole route; card 11 adds a `TestNewDarn_Routing` row walking `on_done` Record-Write → Drift-Baseline → Drift-Await → Darn-Write, and cites `internal/driftengine#TestDriftBaseline` and `#TestDriftAwait` for the drift rows' re-pass behaviour, which those tests already pin, so no end-to-end row repeats them.
- coverage 3, MEDIUM — fixed differently: card 15 adds the row where a resumed session writes the marker again and halts again; the answer reaching the perch is pinned by card 12's `TestDarnPerch_SharesLoomsPerchBarTheAnswerKey` (the record is the perch's answer key), and reaching the resumed writer is stencil prose, which no test asserts (plan spec, "What a card is" 3).
- coverage 4, MEDIUM — fixed as proposed: card 15's `shedrun.StopMarker` sits in `shedrun.RunDir`, as D6 says "in the run's directory".
- coverage 5 + row-validity 8, LOW — fixed as proposed: card 15 reads the halt reason from the session's output pointer `Path`.
- coverage 6, MEDIUM — fixed as proposed: card 9 asserts `env.PreflightBatcher` set on a tapestry seed (`TestWire_DefaultConfig`) and nil on a darn seed (`TestWire_DarnRecipe`).
- coverage 7, MEDIUM — fixed as proposed: card 1 adds rows for an info string holding a backtick and a run of the other character inside an open fence.
- coverage 8, MEDIUM — fixed as proposed: card 3 adds the two-build-rows row.
- coverage 9 + guard-reach 2, MEDIUM — fixed differently: proposed a reaching row; card 8 labels `New`'s non-loom refusal a fail-loud wiring guard, since its one caller `newShed` branches pawl off and card 19 refuses every other non-loom recipe in `armAt` first (guard-reach 2's evidence), so no input reaches it.
- coverage 10, MEDIUM — fixed as proposed: card 20's `TestHubPeersDeps_OverRealHub` reaches each `peerRecipe` branch through the real seam.
- coverage 11, MEDIUM — fixed differently: proposed a row over `wire`'s stat branch, which needs a real fabric; card 17 moves the presence test into `runRecordBullets`/`DescribeTable`, and `TestDescribeSpec_RunRecordsOldestFirstLiveLast` reaches both sides over a temporary directory.
- coverage 12, MEDIUM — fixed as proposed: card 18 asserts a darn seed's `PatternRecordPath` in `TestWire_DarnRecipe`.
- coverage 13, LOW — fixed as proposed: card 18's `TestRecipeKeyedWiring` calls `landingDeps` per recipe for `EscalateJudgedFirst` and `TraceFailure`.
- coverage 14, LOW — fixed as proposed: card 13 adds a nil `DiscussionHistory` row to `TestResolveGateSpec_Rejections`.
- coverage 15, LOW — fixed as proposed: card 5 asserts the run-settings refusal ends in exactly the Spec's `RetiredWayForward` text, and the three reader tests cover pawl.
- coverage 16, LOW — fixed as proposed: card 6's `TestLoadTapestryConfig` covers the empty `plan_judge`/`plan_seed` fallback.
- coverage 17, LOW — fixed as proposed: card 9's `TestPreflight_BatcherCheck` adds the cancelled-context row.
- coverage 18 + guard-reach 1, MEDIUM — fixed differently: proposed keeping the load inside loom's; card 6 now never reads `tapestry.yaml` in loom's `LoadConfig`: `Config.WithTapestry` applies it only where the recipe holds a Plan-Review row (`wire`, `reviewBudget`), so darn, pawl and the code-budget seam never meet a tapestry fault, and D12's plan-perch check stays in the tapestry load.
- row-validity 1, BLOCKING — fixed as proposed: card 15 rebuilds `newDarnFixture` over a temporary directory with a record and support log, and lists `darnFixture`, `darnFixture.session` and `darnSession.Call` for the marker switch.
- row-validity 2, MEDIUM — fixed as proposed: cards 13 and 15 make `CheckRecord`, `CommittedRecord` and the paths required seams `darnWrite.checkSeams` names, filled by the fixture.
- row-validity 3, BLOCKING — fixed as proposed: card 11 lists `RecordWrite.*` in battencli's `intentionallyNil`; cards 13 and 15 list `Darn.CheckRecord` and `Darn.CommittedRecord` in loomcli's and battencli's.
- row-validity 4, MEDIUM — fixed as proposed: card 19 gives a nil `seedRecipe` the tapestry meaning and lists `fakeReviewTarget`.
- row-validity 5, LOW — fixed as proposed: card 9 drops the vacuous resume row, removes the two `shed1.Producers[1]` substitutions, and rests the D10 bound on `wantProducerTable` holding no Tapestry-Preflight row.
- row-validity 6 + forced-edits 1, BLOCKING — fixed as proposed: card 9 lists `TestSequence_FullRunBlocksAtMergeIn` (fixture rewritten onto Preflight then Discussion-Write) and `TestResume_CrashRecoveryRecallsUnconditionally`.
- row-validity 7, LOW — fixed as proposed: card 3 drops the `TestBuild_Errors` row and its valid row calls `checkLoomContract` directly, asserting nil.
- forced-edits 2, MEDIUM — fixed as proposed: card 16 edits `orch-template-role.md`'s decision bullets (no Tapestry-Preflight, no darn writer reading a note, the refusals' own ways forward).
- forced-edits 3, MEDIUM — fixed as proposed: card 5 edits `docs/overview.md`'s shed.yaml paragraph.
- forced-edits 4, MEDIUM — disputed, premise false: `internal/landingshed/deps.go:364-365` documents `BoardEntryPath` recipe-neutrally ("the run's snapshot … listed when it exists. Empty means the run keeps none"), so card 12's change leaves it true.
- forced-edits 5, MEDIUM — disputed, premise false: `TestPRReview_Session` builds its own deps with a snapshot path and pins the prompt listing an existing path; it reads no recipe, so its meaning is unchanged by card 12.
- forced-edits 6, MEDIUM — disputed, premise false: `contracts/stencils/markers_test.go:20-36` `TestStencils_TopLevelMarkers` has no `darn-template-write` row, and the darn writer stencil appears only in `stencils.go:463,570` (registry and role opening), which card 15 does not change.
- forced-edits 7, LOW — fixed differently: proposed a row in loomshed's `TestBatcherRows_StuckSurfacesTheBatcherError`, whose table is loomshed's; card 9's `TestPreflight_BatcherCheck` asserts Preflight logs the row name and batcher error text, in preflightshed where the producer lives.
- forced-edits 8, LOW — fixed as proposed: card 9 edits `internal/shedrun/doc.go`, `internal/hubreconcile/doc.go`, `shed-recipe-spec.md` and `waitForCurrentProducer`'s comment.
- forced-edits 9, LOW — fixed as proposed: card 6 edits `implementer-spec.md`.
- forced-edits 10, LOW — fixed as proposed: card 9 says `tools/tapestryrename` keeps the old names.
- guard-reach 3, LOW — fixed differently: proposed a refusal or exemption; card 19 states `lym loom checklist`'s exemption and its bound (a read-only pane the pre-run never arms, which drives and records nothing).
- guard-reach 4, LOW — fixed differently: proposed noting the dead branch; card 18 deletes `codeReviewOff` and sets `CodeReviewOff` to `codeBudgetZero(l)` for every recipe.
- way-forward 1, MEDIUM — fixed as proposed: card 19 gives a recipe naming no embedded recipe (`beam`) the re-seed way forward, keeping `lym shed` for batten.
- way-forward 2, MEDIUM — disputed, premise false: `armFromSeed` closes `lookup`'s refusal with the generic way forward, pinned by `internal/shedcli#TestArmFromSeed_RefusalEnvelopeKinds`' "unknown recipe on step" row (`internal/shedcli/cli_test.go:173-179`, `wantSuffix: genericWayForward`, `wantWayForwardCount: 1`); card 2 now cites that row and drops its beam row, which repeated it.
- way-forward 3, MEDIUM — fixed as proposed: card 16 names `lym loom start` when the run has no status file.
- way-forward 4, MEDIUM — fixed as proposed: card 5's `RetiredWayForward` text first says to remove the key from the refused file, profile block or `run_settings` object.
- way-forward 5, LOW — fixed as proposed: card 13's guard way forward says the orch runs `lym fabric commit` when a session's call is refused.
- way-forward 6, LOW — fixed as proposed: card 15 also refuses a missing support log before the session, naming `goto --to Record-Write` then `resume`.
- way-forward 7, LOW — fixed as proposed: cards 13, 15 and 16 name `git -C _lym checkout HEAD -- discussion/<file>` for each restore.
- way-forward 8, LOW — fixed as proposed: card 19's review-group refusal says the verb does not apply to a non-loom run.
