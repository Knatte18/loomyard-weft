# Lens review

Every lens fork returned a report: coverage, row-validity, forced-edits, guard-reach and way-forward.
Duplicates are merged, and each line names every lens that raised the finding.

- coverage, MEDIUM — D5 `run_settings` row cannot reach the config load: fixed differently. The proposed row inside `TestPRFindingsVerb` would still pass a plain value. Card 8 instead makes `smallMax` a seam that `prReviewSmallMax(location)` fills from `loomengine.LoadConfig`, and adds `TestPRReviewSmallMax_ReadsThePairsLayers`, built like `TestPullRequestDeps_ParentReviewWantedReadsThePairsLayers`. The same fix covers row-validity 5.
- coverage, MEDIUM — Parent-Align parent commits not listed, no row: fixed differently. The proposed `TestDriftDeps_*` extension was not made, since those tests sit in `loomcli_test` and cannot reach the unexported `driftDeps`. Card 3 now has `loomCLI.wire` set the drift deps' `ReviewBase` from the same `verifyMergeBase` as `Env.ReviewBase`. A `TestWire_ReviewKeysReachTheEnv` row asserts both answer the same commit, and a `TestDriftVerify` row asserts the reads take `ReviewBase`'s answer, not `BranchPoint`'s.
- coverage, MEDIUM — a reused `uncaused` ruling halting again has no row: fixed as proposed. A `TestDriftVerify` row resumes with no new commit, spawns no judge and halts.
- coverage, MEDIUM — command mode with a passing compare and an uncaused file has no halt row: fixed as proposed. The command-mode row now asserts the halt in place of today's Done.
- coverage, MEDIUM — the wake's dirty-after-stop halt goes unreached: fixed as proposed. A `TestPRReview_Wait` row halts dirty, and the next Call reroutes on a clean tree.
- coverage + row-validity + guard-reach, MEDIUM — card 8's no-record refusal and mark-already-held branch have no rows: fixed as proposed. Card 8 spells the no-record refusal with its way forward and adds both rows.
- coverage, MEDIUM — D2 Go-written role commit criterion has no card and no test: disputed, every fix worse. No test can reach a site that does not exist, and making Go write a role's commit adds scope the record does not license.
  - The commit sites read this session are `gitrepo`'s `StageAndCommit` and `StageAllAndCommit` callers in `fabricengine` (`recordsgit.go:268`, `bolt.go:68`, `stencilcommit.go:75`, `commitweftpaths.go:66`), all records, board or stencil commits.
  - The other sites are `landingshed/finalize.go:444`, the landing merge after review, and `fabricengine/commit.go:225`, reached only through the `lym fabric commit` verb with the fixed message `fabricengine.DefaultCommitMessage` (`fabriccli/weft_verbs.go:203`).
  - The overview's Shared Decision records this, and card 11's fragment carries D2 for every session-made commit.
- coverage, LOW — no-mark re-step keeps the count: fixed as proposed (`TestPRReview_Call` row).
- coverage + row-validity, LOW/MEDIUM — old `review_input` generation record and `FullGeneration` true have no rows: fixed as proposed (`TestBouncer_Amendment` rows).
- coverage, LOW — D6 note content and Tier 2 off have no rows: fixed as proposed. `TestPRFindingsVerb` asserts the note text and an empty friction directory.
- coverage, LOW — unresolvable refs in card 1 have no rows: fixed as proposed. Both fabric tests gain an unresolvable-ref row.
- coverage, LOW — reroute cap carries no cited test: fixed as proposed. Card 9 names `TestRun_RerouteCapHaltsOnFourthRerouteAndResumeGrantsThreeMore` under Uses and in its Intent.
- forced-edits + row-validity, BLOCKING — the retyped review-input seam has unlisted callers: fixed as proposed. Card 7 now edits `withReviewInput`, `amendmentInputRow`, `reentryInputRow`, `TestWire_DarnRecipe` and `TestWire_ReviewKeysReachTheEnv`, and its Intent says how each follows. The `ReviewInputErrorIsReturnedWithNothingArchived` row sits in the listed `TestBouncer_Amendment`.
- forced-edits, MEDIUM — shedadapters `doc.go` describes the old seam: fixed as proposed. It moves to card 7's Edit, and the Intent says what it gains.
- forced-edits, LOW — `nilLegal` key rename: fixed as proposed. Card 7 states `"ReviewInput"` becomes `"ReviewInputs"`.
- row-validity, BLOCKING — `prReviewTestDeps` must fill `ShedStatus`: fixed as proposed. Card 9 lists it and gives it a scripted status, empty history by default.
- row-validity, MEDIUM — drift fixture lacks the new seams and file rulings: fixed as proposed. Card 3 now edits `verifyFixture`, `newVerifyFixture` and `verifyFixture.judgeArrives`, and its Intent says what each gains, the trailer-only commit included.
- row-validity, MEDIUM — `templateDefaults` must carry the new default: fixed as proposed (card 4 edits `internal/loomengine#templateDefaults`).
- row-validity, LOW — the `TestRecords` old-record row repeats existing coverage: fixed as proposed. The row is dropped, `TestRecords` moves to card 2's Uses, and the Intent names `TupleAddedToJudged` as the existing cover of the old-record criterion.
- row-validity, LOW — `TestRecipe_PRReviewReroutesToCodeBouncer` should be a table row: disputed, premise false.
  - `TestRecipe_WebsterReroutesToPlanReview` (`internal/loomrecipe/recipe_test.go:424`) and `TestRecipe_FinalizeDeclaresNoReroute` (`:399`) each pin another row's edge, so no existing test covers the PR-Review edge.
  - The new test is itself one table over the three recipes.
  - Folding the others into it would rename existing tests outside this task's scope.
- row-validity, LOW — the wait row cannot count a session stop: fixed as proposed. `newWaitWorld` takes a `fakePRSessions`, listed under card 9's Uses.
- row-validity, LOW — no row pins the ordering of no-match and uncaused halts: fixed as proposed. Card 3 states one Stuck carrying both reasons, no-match first, and adds the row.
- way-forward, MEDIUM — a fix-here re-run after a failed commit double-counts: fixed differently. A commit-only retry verb was not named, since no verb is known to commit the pull-request record alone.
  - Card 5 adds `CountedSets`, the hashes of counted findings files, which the clear empties.
  - Card 8 counts a file once, so "run the verb again" progresses for fix-here and send-back alike.
  - A failing `commit` seam row pins this.
- way-forward, MEDIUM — the parse refusal names a path absent from a hub worktree: fixed as proposed. The message spells `<stencilsDir>/burler/burler-review-format.md` from the told stencils directory.
- way-forward, MEDIUM — `sendBackEdge`'s error branch is unspecified: fixed as proposed. Card 8 adds a read-failure envelope with its way forward and the record unchanged, and a row reaches it.
- way-forward, LOW — the missing-fragment error names no way forward: fixed as proposed. Card 6 wraps it, naming both stencils and `lym stencil diff` then `lym stencil sync`, and its row asserts that.
- way-forward, LOW — the reroute-cap halt cycles for a finding Code-Review keeps approving: disputed, every fix worse.
  - Loom's cap way forward is row-neutral by test (`internal/loomcli#TestLoomRerouteCapWayForward_IsRowNeutral`, `internal/loomcli/wiring_test.go:1264`), and a PR-Review-specific clause breaks that test.
  - D4's Bound names the cap's own way forward as the bound, and the parent keeps `lym loom goto` and `run_settings`.
- guard-reach, BLOCKING — a send-back after a zero-allowance amendment falls to the `n == 0` seed path: fixed as proposed. Card 7's `reenterUnchanged` starts the full generation itself when a full input is kept, and a `TestBouncer_Amendment` row covers the zero-allowance case.
- guard-reach, MEDIUM — the listing never runs when scope is off, deferred, waived, unrecorded or never counted: fixed as proposed, by its second option. Card 3 states and bounds the skip, with D1's rubric bullet as the check there, and `doc.go` carries it. Running the listing before those returns was not chosen, since the judge request needs the seal those passes lack.
- guard-reach, MEDIUM — a mark with no own history entry has no row: fixed as proposed (`TestPRReview_Call` row).
- guard-reach, LOW — halts before the listing leave the verdict without loosened checks: fixed as proposed. Card 3 states that those halts record a check with no `Loosened` and that the resume reaches the listing.
