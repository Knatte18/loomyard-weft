# Card 18 — merge-in-row

**Create:**
- `plan:internal/loomshed#NameMergeIn` -> `const NameMergeIn = "Merge-In"`
- `plan:internal/loomshed#NameMergeBouncer` -> `const NameMergeBouncer = "Merge-Bouncer"`
- `plan:internal/loomshed#NameMergeBurlerReview` -> `const NameMergeBurlerReview = "Merge-Burler-Review"`
- `plan:internal/loomshed#NameMergeBurlerFix` -> `const NameMergeBurlerFix = "Merge-Burler-Fix"`
- `plan:internal/shedrecipe#mergeInEntry` -> `func mergeInEntry(name string, cfg Config, env Env) (shedengine.ShedProducer, error)`

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#simpleEntryCases`
- `internal/shedrecipe#validLandingDeps`
- `internal/loomshed#InterruptPolicies`
- `internal/loomrecipe#applyDarnReviewBudget`
- `internal/landingshed#Publish`
- `internal/landingshed#NewPublish`
- `internal/landingshed#Publish.Call`
- `internal/landingshed/doc.go`
- `internal/landingshed#TestNewPublish_Refusals`
- `internal/landingshed#TestPublish_StuckBeforePullRequest`
- `internal/landingshed#TestPublishVerify_Proceeds`
- `internal/landingshed#TestPublishVerify_HaltsOnVerifyResult`
- `internal/landingshed#TestPublishVerify_PublishVerifyCommand`
- `internal/landingshed#TestPublishVerify_DirtyTreeHalts`
- `internal/landingshed#TestPublishVerify_HardErrorsAreReturned`
- `internal/landingshed#TestPublishVerify_FailureRecord`
- `internal/landingshed#TestPreMerge_ClearsOwnParkedMergeIn`
- `internal/landingshed_test#TestPublish_MergesInCleanlyBeforeCreatingPullRequest`
- `internal/landingshed_test#TestPublish_RejectedPushNamesRemoteTipThenResumesAfterMerge`
- `internal/loomcli#isLandingProducer`
- `internal/loomcli#ownMergeInLeftover`
- `internal/loomcli#TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals`
- `internal/loomcli#perchMaxBounces`
- `internal/loomcli#perchAutoAccept`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomCLI.wireLightweight`
- `internal/loomcli#TestWire_Real_PublishRejectedPushNamesRemoteTip`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#wantDarnProducerTable`
- `internal/loomrecipe#wantSequenceOrder`
- `internal/loomrecipe#testEnv`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/loomrecipe#TestRecipe_PublishRoutesThroughPRGate`
- `internal/loomrecipe#TestNew_PublishAndFinalizeAreRealProducers`
- `internal/loomrecipe#TestCoverageGuard_PublishAndFinalizeRowNamesMatchTheirProducerIdentity`
- `internal/loomrecipe#TestShippedRecipe_SeamsWiredOnPlanBouncerOnly`
- `internal/loomrecipe#reviewSegmentBouncers`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestDarnRouting`
- `contracts/specs/refusal-spec.md`

**Rename:**
- `internal/loomrecipe#TestSequence_FullRunBlocksAtPublish` -> `plan:internal/loomrecipe#TestSequence_FullRunBlocksAtMergeIn`

**Delete:**
- `internal/landingshed#TestPublishVerify_ResolverStuck`
- `internal/landingshed#TestPublish_CommitStatus`

**Uses:**
- `plan:internal/landingshed#NewMergeIn`
- `internal/landingshed#ReadMergeRecord`
- `plan:contracts/stencils#LoomRubricMergeReview`

**Intent:**
Put Merge-In and the Merge-Review perch into the recipes and take the merge-in out of Publish (D1, D2).

Loom's order becomes `Webster-Bouncer → Describe → Merge-In → Merge-Bouncer → Publish → PR-Gate → Finalize`.
- `Describe` hands to `Merge-In` (engine `MergeIn`), whose Done goes to `Merge-Bouncer`.
- `Merge-Bouncer` (engine `Bouncer`, segment `Merge-Review`) has `on_stuck: Merge-Burler-Review` and `on_done: Publish`.
  Its config: `run_subdir: merge`, `artifact_paths: [_lyx/discussion/decision-record.md]`, `rubric_stencil: loom-rubric-merge-review`, `approve_seam: merge`, `skip_seam: merge-overlap`, `blocking_escalates: true`, `carry_over: Merge-Review` and `color_segment: merge`.
- `Merge-Burler-Review` and `Merge-Burler-Fix` close the ring in the shape Webster-Review uses, with `run_subdir: merge` and `color_segment: merge`.
  The review row's profile has a target with instructions only, saying the subject is the interaction between the parent commits merged in and the task's own change as the rubric's merge section defines them.
  Its fasit is the decision record, with instructions to read `PATTERN.md` and the `pattern/` files the merge section names.
  It runs `rubric_stencil: loom-rubric-merge-review`, `fix-scope: source` and `tool-use: true`, with no cluster fan.
  The fix row has `review_row: Merge-Burler-Review`, the `verify` gate with 3 attempts and `fix-scope: source`.
- `Publish`, `PR-Gate` and `Finalize` declare `on_reroute: Merge-In`; card 20 makes them use it.
Darn's order becomes `… → Describe → Merge-In → Publish → PR-Gate → Finalize`, with no merge perch: its `Merge-In` hands straight to `Publish`, and its three landing rows declare `on_reroute: Merge-In`.
`applyDarnReviewBudget` routes a 0 budget's `Darn-Write` Done to `Merge-In` instead of `Publish`, so the merge-in still runs under the lock.
Both recipe headers explain the new rows, and that a run parked on a row whose name or edges changed is restarted, not migrated.

Wiring:
- `registry` gains `"MergeIn": mergeInEntry`, `publishEntry`'s twin over `landingshed.NewMergeIn(env.Landing)`.
  `simpleEntryCases` gains its row, and `validLandingDeps` fills the seams `NewMergeIn` requires.
- `loomshed` gains the four `Name*` constants, pinned to the rows by the existing coverage guard.
  `InterruptPolicies` lists all four as reinvoke: Merge-In spawns only the conflict session the resolver reattaches, and the perch's rows probe for a live session.
- In loomcli, `perchMaxBounces` and `perchAutoAccept` map `Merge-Bouncer` to `merge_max_bounces` and `merge_auto_accept`.
  `wire` gives `Merge-Bouncer` the Webster-Review judge models and the two Burler rows the Webster-Review round models: the merge review is the other committed-diff review.
- `wire` and `wireLightweight` tell `ShedPaths.RerouteCapWayForward` as `run "lyx loom resume" in the task worktree once the parent branch has settled`.
- `isLandingProducer` names Merge-In and Finalize, the two rows that now abort and redo their own parked merge-in, so `lyx loom start` and `lyx loom resume` let a loop killed during Merge-In's conflict session resume through Merge-In's clearing.

Publish keeps `publish_verify`, the push and the pull request, and stays a no-op for a parent outside `require_pr_to_base`.
- It no longer commits the status file, clears a parked merge-in, merges the parent or runs the plan verify.
- `Publish` loses its resolver and pre-merge fields, and `NewPublish` stops building the resolver and no longer requires `OpenFabric`.
  `Publish.Call` no longer calls `noteMergeCommit`, which card 20 deletes.
- Its failure record takes `merge_commit` from the merge record.
- It removes the record after `publish_verify` passes, or at once when no `publish_verify` is set.
Merge-In removes it after its plan verify passes.
The failure record therefore names `Merge-In` for the plan verify and `Publish` for `publish_verify`.

`landingshed/doc.go` moves the merge-in, the clean checks, the conflict session and the plan verify from Publish's paragraphs to a Merge-In paragraph.
`refusal-spec.md` renames the landing rows Publish shared with Finalize to Merge-In and Finalize, splits `Publish: verify failed` into Merge-In's plan verify and Publish's `publish_verify`, and names Merge-In in the start/resume rows.

Tests:
- The recipe and darn tables, the sequence order and the coverage, routing, real-producer and approve-seam tests follow the new rows.
- `testEnv` and `buildSequenceFixture` give Merge-In a temporary lock store and fake tip reads.
  The fixture never drives a genuine merge, so its fake tips report the local parent ahead of its upstream and Merge-In halts before any merge.
  The sequence test, renamed `TestSequence_FullRunBlocksAtMergeIn`, asserts the row order through Describe and the blocked halt at Merge-In, and keeps its plan-approval guard.
- The Publish tests whose subject was its merge-in or plan verify assert the same behaviour at Merge-In, or are deleted where card 12's tests cover it.
  `TestPreMerge_ClearsOwnParkedMergeIn` runs its Publish cases against Merge-In.
- The mid-merge start test names Merge-In.

**ImpactSummary:** Publish loses its merge-in and plan verify, and both recipes gain the Merge-In row with loom's Merge-Review perch; the recipe, wiring, registry and landing tests are updated to the new row order.

**Commit:** 18: loom, darn: Merge-In row and the Merge-Review perch ahead of Publish
