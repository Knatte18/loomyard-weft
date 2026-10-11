# Card 4 — split-burler-rows

**Create:**
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#burlerFixEntry`
- `internal/shedrecipe#burlerFixProfile`
- `internal/shedrecipe#burlerRowModels`
- `internal/loomshed#NameDiscussionBurlerReview`
- `internal/loomshed#NameDiscussionBurlerFix`
- `internal/loomshed#NamePlanBurlerReview`
- `internal/loomshed#NamePlanBurlerFix`
- `internal/loomshed#NameWebsterBurlerReview`
- `internal/loomshed#NameWebsterBurlerFix`
- `internal/loomshed#NameDarnBurlerReview`
- `internal/loomshed#NameDarnBurlerFix`
- `internal/loomrecipe#burlerReviewType`
- `internal/loomrecipe#burlerFixType`
- `internal/loomcli#probingHalfRunner`
- `internal/loomcli#probingHalfRunner.Review`
- `internal/loomcli#probingHalfRunner.ProbeReview`
- `internal/loomcli#probingHalfRunner.ResumeReview`
- `internal/loomcli#probingHalfRunner.Fix`
- `internal/loomcli#probingHalfRunner.ProbeFix`
- `internal/loomcli#probingHalfRunner.ResumeFix`
- `internal/loomcli#TestSmokeBurlerReview_AttachesToALiveReviewerInsteadOfRespawning`
- `internal/shedrecipe/entries_burlerhalves_test.go`

**Delete:**
- `internal/shedrecipe#burlerRoundEntry`
- `internal/shedrecipe/entries_burler_test.go`
- `internal/loomrecipe#burlerType`

**Edit:**
- `internal/shedrecipe#burlerRoundProfile`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe#TestGateCapableEntries_RejectLeftoverOldKeys`
- `internal/shedrecipe#newTestEnv`
- `internal/shedbuild#engineMinimalConfig`
- `internal/testkit/envkit#FullEnv`
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `internal/loomshed#InterruptPolicies`
- `internal/loomshed#TestInterruptPolicyFor`
- `internal/loomrecipe#darnVerifyGatedRows`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#wantDarnProducerTable`
- `internal/loomrecipe#newLoomBurler` -> `func newLoomBurler(t *testing.T) *shedfake.BurlerHalfRunner`
- `internal/loomrecipe#newEscalationShed` -> `func newEscalationShed(t *testing.T) (*shedengine.Shed, string, *shedfake.BurlerHalfRunner, func() shedengine.Status)`
- `internal/loomrecipe#testEnv`
- `internal/loomrecipe#buildSequenceFixture`
- `internal/loomrecipe#deriveSequenceOrder`
- `internal/loomrecipe#TestSequence_FullRunBlocksAtPublish`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#assertReviewBudget`
- `internal/loomrecipe#TestBouncerRunSubdir`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestDarnVerifyBudget`
- `internal/loomrecipe#TestDarnRouting`
- `internal/loomrecipe#TestEscalation_BudgetContinueGrantsOneRoundThenAccept`
- `internal/loomrecipe#TestBounceRouting_StuckContinuesAtDeclaredTarget`
- `internal/loomrecipe#TestBounceRouting_BudgetExhaustionBlocks`
- `internal/loomrecipe#clusterExcludesMismatches`
- `internal/loomrecipe#TestClusterExcludesMismatches_Cases`
- `internal/loomrecipe#gateQuiescenceFailureMessage`
- `internal/loomrecipe#gatedRowFanViolations`
- `internal/loomrecipe#TestNoGatedRowAuthorizesForkSubagents`
- `internal/loomrecipe#assertOverlayBurlerCommitSeams`
- `internal/loomrecipe#TestOverlayBurlerCommitSeams_RejectsBadShapes`
- `internal/loomrecipe#overlaySeamFixture`
- `internal/loomrecipe#overlaySeamFixtureNoBouncer`
- `internal/loomrecipe#overlaySeamFixtureStraddling`
- `internal/loomrecipe#overlaySeamFixtureInstructionsOnly`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#statusCommitPathspec`
- `internal/loomcli#planVerifySource`
- `internal/loomcli#loomCLI.validateDiscussionCmd`
- `internal/loomcli#loomCLI.validatePlanCmd`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/loomrecipe#TestNew_ShapeMatchesRecipe`
- `internal/shedadapters/burler.go`
- `internal/shedadapters/burler_test.go`
- `contracts/stencils/orch/orch-template-role.md`
- `contracts/stencils#wordingClaims`
- `contracts/specs/refusal-spec.md`
- `docs/overview.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:**
D1 and D5: every perch's single `BurlerRound` row becomes a review row and a fix row in the same segment, in both recipes, wired to card 3's producers.

Recipes: in `loom-recipe.yaml` (Discussion-Review, Plan-Review, Webster-Review) and `darn-recipe.yaml` (Darn-Review), each `<Perch>-Burler` row becomes `<Perch>-Burler-Review` (engine `BurlerReview`) then `<Perch>-Burler-Fix` (engine `BurlerFix`), both in the perch's segment.
Routing: the Bouncer's `on_stuck` names the review row, the review row's `on_stuck` names the fix row, the fix row's `on_stuck` names the Bouncer, and each row's `on_done` names the same row as its `on_stuck`, the guard against an empty `on_done`, since neither row ever returns Done.
A Done by any row of a segment ends its bounce episode, so the hand-offs between the halves are Stucks.
The review row carries `run_subdir`, `profile` (target, fasit, rubric or rubric_stencil, fix-scope, tool-use, as the round row carried it) and optional `timeout_s`; the fix row carries `run_subdir`, `review_row` (its partner's name, for the missing-acceptance way forward), the `gates` key moved unchanged from the round row, `profile` holding only `target` and `fix-scope`, and optional `timeout_s`.
Each recipe header's routing and budget paragraphs name the two rows, and states that an in-flight run parked on a retired `*-Burler` row is restarted, not migrated, as it already says for earlier retired rows.

`shedrecipe`: `registry` maps `BurlerReview` to `burlerReviewEntry` and `BurlerFix` to `burlerFixEntry`, and drops `BurlerRound` with `burlerRoundEntry`.
`burlerReviewEntry` keeps `burlerRoundEntry`'s run directory, profile, rubric stencil, cluster fan (`Env.RowClusterFans`), timeout and model handling, takes no `gates` key (its gate is the engine's review-parse entry), and returns `shedadapters.NewBurlerReviewProducer`; `burlerRoundProfile`'s error labels name `BurlerReview`.
`burlerFixEntry` reads `run_subdir`, `review_row`, `gates` (through `resolveRowGateSpec`), `profile` through `burlerFixProfile` (keys `target` and `fix-scope` only) and `timeout_s`, takes no fan, and returns `shedadapters.NewBurlerFixProducer`.
`burlerRowModels` is the shared models pick: the row's `Env.RowReviewModels` entry, else `Env.ReviewModels`; each row takes its own half of the pick.
`Env.Burler` becomes a `shedadapters.BurlerHalfRunner`, and the `Env` field docs name the review and fix rows where they say `BurlerRound`, and `DarnVerifyAttempts`'s doc names `Darn-Burler-Fix` in place of `Darn-Burler`; `bouncerEntry`'s comments (and its file header) name the review row as the Bouncer's partner.
`entries_burlerhalves_test.go` replaces `entries_burler_test.go` and covers both entries: profile mapping, rubric stencil, models fallback, timeout, the relative-path exception, strict unknown keys (the fix entry refusing fasit, rubric and cluster-fan), run directory creation, gate config on the fix row, construction failures and the file-set error labels.
The `internal/shedrecipe` row of `test-budget.yaml` follows the package's new count, and the `internal/loomcli` row rises by the new smoke test, which the budget counts under every build tag.
`minimalEntries`, `TestNames`, `TestGateCapableEntries_RejectLeftoverOldKeys` and `newTestEnv` move to the two entries and the new fake, and `shedbuild`'s `engineMinimalConfig` carries a minimal config for each of the two engines in place of `BurlerRound`'s.

`loomshed`: eight `Name*BurlerReview` and `Name*BurlerFix` constants name the new rows, the recipe coverage guard pins them, and `InterruptPolicies` maps all eight to `InterruptPolicyReinvoke`, since both producers probe and resume a live half rather than spawn a second one.
Every reference to the four `Name*Burler` constants moves to the new ones here, and card 10 deletes the four.
`shedadapters`' `burler.go` drops its compile-time assertions that `*burlerengine.Engine` satisfies `BurlerRunner` and that `*BurlerProducer` is a `shedengine.ShedProducer`, since the engine is now wired through `BurlerHalfRunner`, whose own assertion card 3 added; `burler_test.go` drops its assertion that `shedfake.BurlerRunner` satisfies `BurlerRunner` with them.
`loomrecipe`: `darnVerifyGatedRows` names `NameDarnBurlerFix`, since the verify gate moved to the fix row.
`loomcli`: `loomCLI.wire` keys `RowReviewModels` by both rows of each perch (the Darn rows take Webster-Review's models) and `RowClusterFans` by each Bouncer and its review row; `Env.Burler` is still the burler engine, which satisfies the new seam.
`planVerifySource`'s Publish way forward becomes `run "lyx loom goto --to Webster-Burler-Review", then "lyx loom resume", in the task worktree; ...`, since a goto into the fix row with no accepted review is refused; the refusal-spec's `Publish: verify failed` row, `orch-template-role.md` and the `wordingClaims` entry pinning the old text change with it, and `TestLandingDeps_VerifyCommandReadsPlanAtCallTime` and `TestWire_DarnRecipe`, which assert it, follow.
`validateDiscussionCmd` and `validatePlanCmd`'s help names the fix rows' gates; `statusCommitPathspec`'s comment names the two entries.
The refusal-spec rows `Webster-Burler verify gate exhausted`, `gate slot: gate.yaml absent or invalid` and `producer re-run: superseded strand not removed` name the fix rows and a Burler review or fix row's half.
`docs/overview.md` names the split rows where it names `Discussion-Burler`, `Plan-Burler` or `Darn-Burler`.

Tests follow the new graph: the producer tables gain both rows with their engines and producer types (`burlerReviewType`, `burlerFixType`), and `TestNew_ShapeMatchesRecipe` checks that each Bouncer's `OnStuck` is a review row; `newLoomBurler` and `newEscalationShed` return the half fake, whose `DuringReview` writes the review and `DuringFix` the fixer report; a clean run's sequence holds, per segment, the Bouncer's seed Stuck, the review row's Stuck, the fix row's Stuck and the Bouncer's Done, and counts one review and one fix call per segment; the escalation test counts the hand-backs on both rows as budget-exempt; the bounce-routing tests name the review row as the Bouncer's bounce target; the progress test puts the Bouncer and both rows on one step; the review-budget assertion counts the split segments' rows; the cluster-exclusion check pairs a Bouncer with its review row; the gate-quiescence rule allows a fan on a `BurlerReview` row, which carries no `gates` key, and gates on a `BurlerFix` row, which never forks; the overlay-seam guard walks `BurlerFix` rows.
The tmux smoke test starts one live shuttle run declaring a round's review, calls `BurlerReviewProducer.Call` over the same run directory and asserts it resumed that run rather than starting a second reviewer, through `probingHalfRunner`, which implements the half seam over the real shuttle runner and fails the test on any spawn; it and `TestSmokeBurlerReview_AttachesToALiveReviewerInsteadOfRespawning` land in `smoke_attachprobe_test.go` beside `probingBurlerRunner`, its three methods and `TestSmokeBurlerRound_AttachesToALiveRoundInsteadOfRespawning`, which still pin `BurlerProducer` until card 10 deletes all of them.

**ImpactSummary:** the shipped loom and darn graphs, the row-name constants, the shedrecipe registry and `Env.Burler`'s type change together; `BurlerProducer`, the old fake and the old row-name constants are left referenced by no production code, only by their own tests, for card 10 to delete.

**Commit:** 4: loom: split every Burler row into a review row and a fix row
