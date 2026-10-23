# Card 22 — recipe-tails

**Create:**
- `plan:internal/loomshed#NameCodeBouncer` -> `const NameCodeBouncer = "Code-Bouncer"`
- `plan:internal/loomshed#NameCodeBurlerReview` -> `const NameCodeBurlerReview = "Code-Burler-Review"`
- `plan:internal/loomshed#NameCodeBurlerFix` -> `const NameCodeBurlerFix = "Code-Burler-Fix"`
- `plan:internal/loomshed#NamePRReview` -> `const NamePRReview = "PR-Review"`
- `contracts/stencils/loom/loom-rubric-code-review.md`
- `plan:contracts/stencils#LoomRubricCodeReview` -> `var LoomRubricCodeReview []byte`

**Delete:**
- `contracts/stencils/loom/loom-rubric-webster-review.md`
- `contracts/stencils#LoomRubricWebsterReview`
- `internal/shedrecipe#publishEntry`
- `internal/shedrecipe#prGateEntry`
- `internal/shedrecipe#prAlignEntry`
- `internal/shedrecipe/entries_prrework.go`
- `internal/shedrecipe/entries_prrework_test.go`
- `internal/shedrecipe#TestPublishEntry_LandingRejected`
- `internal/shedrecipe#TestPRGateEntry`
- `internal/loomrecipe#TestRecipe_PublishRoutesThroughPRGate`

**Edit:**
- `contracts/recipes/loom-recipe.yaml`
- `contracts/recipes/darn-recipe.yaml`
- `contracts/stencils#entries`
- `contracts/stencils#TestLoomRubrics`
- `internal/loomcli#TestReviewRange_ReworkBranchListsOnlyOwnCommitsAfterHead`
- `internal/shedrecipe#registry`
- `internal/shedrecipe#minimalEntries`
- `internal/shedrecipe#simpleEntryCases`
- `internal/shedrecipe#TestNames`
- `internal/shedrecipe_test#coverageGuardAllowedUnreachableEngines`
- `internal/loomshed#InterruptPolicies`
- `internal/loomrecipe#applyDarnReviewBudget`
- `internal/loomrecipe#wantProducerTable`
- `internal/loomrecipe#wantDarnProducerTable`
- `internal/loomrecipe#loomReviewSegments`
- `internal/loomrecipe#assertReviewBudget`
- `internal/loomrecipe#TestBouncerRowsCarryOverTheirOwnSegment`
- `internal/loomrecipe#TestBouncerRunSubdir`
- `internal/loomrecipe#TestCoverageGuard_PublishAndFinalizeRowNamesMatchTheirProducerIdentity`
- `internal/loomrecipe#TestDarnVerifyBudget`
- `internal/loomrecipe#TestEntryNumbers`
- `internal/loomrecipe#TestEscalation_PerchBudgetOneWithAutoAccept`
- `internal/loomrecipe#TestIsFinalizeRow`
- `internal/loomrecipe#TestNewDarn_Routing`
- `internal/loomrecipe#TestNew_MissingLandingClosureReturnsError`
- `internal/loomrecipe#TestNew_PublishAndFinalizeAreRealProducers`
- `internal/loomrecipe#TestReviewBudget_AppliedByNewAndRouting`
- `internal/loomrecipe#TestReviewBudget_NegativeRefused`
- `internal/loomrecipe#TestRouting_ProgressAt`
- `internal/loomrecipe#TestShippedRecipe_SeamsWiredOnPlanBouncerOnly`
- `internal/loomrecipe#TestShippedRecipes_FanSettingsAgreeWithinSegment`
- `internal/loomrecipe#TestShippedRecipes_LanguageSkillsOnCodeRounds`
- `internal/loomrecipe#TestZeroBudget_BouncerSettlesAtOnce`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomPerches`
- `internal/loomcli#loomSegmentPerches`
- `internal/testkit/envkit#FullEnv`
- `contracts/specs/implementer-spec.md`

**Extends:**
- `internal/loomrecipe/doc.go`

**Intent:** Rebuild both recipes' tails (D1, D5, D9) and retire the PR and Merge-Review rows from them, so an in-flight run on a removed or renamed row is restarted, not migrated, as the board's landing note arranges.

The loom recipe after Webster becomes `Webster` → `Merge-In` → `Code-Bouncer` ring → `Describe` → `PR-Review` → `Finalize` → `Friction-Reflect`.
Webster's `on_done` names `Merge-In`; Merge-In's names `Code-Bouncer`; the Code-Review ring is the Webster-Review ring renamed: rows `Code-Bouncer`, `Code-Burler-Review` and `Code-Burler-Fix`, segment `Code-Review`, `role_prefix` and `run_subdir` `code`, `carry_over: Code-Review`, rubric `loom-rubric-code-review`, settings `loom.code_reviewer_fan`, `loom.code_fixer_fan` and `loom.code_auto_accept`.
Code-Bouncer's `artifact_paths` and Code-Burler-Review's `fasit.paths` name the decision record and the plan directory, its answer key, and Code-Bouncer's `on_done` names `Describe`.
Describe's `on_done` names `PR-Review`, a row of engine `PRReview` with `config: {pr_review_setting: loom.pr_review}` whose `on_done` names `Finalize`; Finalize keeps `on_reroute: Merge-In` and `on_done: Friction-Reflect`, and neither Merge-In nor Finalize carries `on_stuck`.
The rows Merge-Bouncer, Merge-Burler-Review, Merge-Burler-Fix, Publish, PR-Align, PR-Gate and PR-Rework go.

The darn recipe becomes `Preflight` → `Darn-Write` → `Darn-Bouncer` ring → `Describe` → `PR-Review` → `Finalize` → `Friction-Reflect`, with no Merge-In.
Darn-Write loses its `segment: PR-Review` and its `on_reroute`; Describe's `on_done` names `PR-Review`, whose setting is `darn.pr_review`; Finalize's `on_reroute` names `Darn-Bouncer`.
`applyDarnReviewBudget`'s budget-0 rewrite routes Darn-Write's `on_done` to `PR-Review` and Finalize's `on_reroute` to `PR-Review`, so neither the perch nor Describe runs and the writer's own description stands.
Each recipe's header comment is rewritten to the new tail: which rows escalate, the reroute edges and their cap, and that a run parked on a removed row is restarted.

The new row names are `loomshed` constants beside the old ones, which stay until the readers of old runs move off them; `InterruptPolicies` maps the new rows and drops the removed ones.
The Webster-Review rubric file becomes `loom-rubric-code-review`, registered in `entries` under its new name, with its title and segment wording renamed.
Its measuring-stick section names the decision record and the plan as its answer key, its Do-not-flag rule names both as the measuring stick and never the subject, and its Also-flag section reviews the code in general, correctness beyond plan compliance, as well as against both (D5); its review range and inputs change with the merge-base and review-input cards.
The registry drops `Publish`, `PRAlign`, `PRGate` and `PRRework`, whose entry functions and tests go, and `PRReview` leaves the coverage guard's allowlist now that a recipe names it.
loomcli's perch tables key `Code-Bouncer` and `Code-Review`, and `wire`'s rework review subdirectories name `code` in place of `webster`.
`envkit.FullEnv` keys its loom segment settings `Code-Review` in place of `Webster-Review`, so every test that builds the loom recipe over it still finds a budget for each review segment.
`implementer-spec.md`, which pins the edges every implementer builds the Webster row with, names `Merge-In` as Webster's `on_done` and the Code-Review perch after Merge-In.
The loom and darn row tables, sequence, routing and budget tests follow the new rows; the loom recipe's routing test pins D1's order and edges, the darn one D9's, and the budget-0 darn test the rewrite to PR-Review on both edges.

`loomrecipe`'s `doc.go` describes the new tails and the darn budget-0 rewrite.

**ImpactSummary:** Both embedded recipes and their row, routing and registry tables change together; the landing producers of the removed rows stay built but unreached until their deletion card.
