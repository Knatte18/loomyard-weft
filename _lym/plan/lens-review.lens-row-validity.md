```yaml
lens: row-validity
outcome: reported
```

Lens: row-validity, checking each test row the plan names (reach, fails-if-wrong, needed).

**Findings**

1. **BLOCKING** — Card 9, seam wiring. `checkWiring` will require the new `ShedStatus`, but no listed helper fills it in the PR-Review test deps.
   - `internal/landingshed#prReviewTestDeps` builds the `Deps` for every PR-Review test, and the card does not list it.
   - `newPRReviewFixture` (`prreview_test.go:701`) and `newWaitWorld` (`prreview_wait_test.go:65`) both call it.
   - Unless the card lists `prReviewTestDeps` and gives it a scripted status, every `TestPRReview_*` test fails on "Deps.PullRequest.ShedStatus is not set".

2. **BLOCKING** — Card 7, retyped seam. Changing `ReviewInput` to `ReviewInputs` breaks test helpers no card lists:
   - `internal/shedadapters#withReviewInput` assigns `cfg.ReviewInput` with the old signature (`bouncer_amendment_test.go:79-81`).
   - `amendmentInputRow` reads `generation().ReviewInput` (`:187-196`).
   - The `ReviewInputErrorIsReturnedWithNothingArchived` row sets `cfg.ReviewInput` (`:307`).
   - `bouncer_reentry_test.go`, `loomcli/wiring_test.go` and `wiring_darn_test.go` also reference `ReviewInput`.
   - The card lists none of these, so the package tests fail to compile.

3. **MEDIUM** — Card 3, `TestDriftVerify` rows. The rows need fixture changes the card does not list:
   - `verifyFixture` / `newVerifyFixture` (`verify_test.go:121`, `:165`) have no fields for `ChangedFiles`, `CommitMessages` or `ReviewBase`.
   - The fake judge `verifyFixture.judgeArrives` writes only cell and case rulings. To write `files` rulings (`none`, `loosened`, `caused`, `uncaused`) it needs a new field and a change.
   - The trailer-only-commit row needs the fixture to change the `CommitMessages` answer together with `moveHead`.
   - These members must be listed under Edit. Without them, the uncaused, caused and reuse rows cannot reach their branches.

4. **MEDIUM** — Card 4, `TestLoadConfig_Loads`. Its "template defaults" subtest asserts only the keys in `internal/loomengine#templateDefaults` (`config_test.go:168-178`). "Its template row asserts 3" therefore needs a `templateDefaults` entry `PRReviewSmallMax: 3`, and the card does not list that var.

5. **MEDIUM** — Card 8, row "`smallMax` read from a local layer holding `pr_review_small_max: 1`". The row cannot reach a layer read.
   - `prFindingsVerb` takes `smallMax` as a plain deps value, and `TestPRFindingsVerb` drives the verb over fakes. Only the command's wiring loads loom config, and no planned row runs it.
   - As written, the row tests nothing about layers. The D5 criterion that `run_settings` overrides the key is left unreached.
   - Fix: test the reader the way `TestPullRequestDeps_ParentReviewWantedReadsThePairsLayers` (`loomcli`) does, as a row over the command's config read. Or name a deps constructor that the row calls.

6. **MEDIUM** — Card 8, prescribed refusal and branch with no reaching row.
   - Step 3, the refusal of a run with no pull-request record, has no row in `TestPRFindingsVerb`, against `PATTERN-refusal-way-forward`.
   - Step 4, "a record already holding a send-back mark answers the send-back again, writing nothing", has no row either.

7. **MEDIUM** — Card 7, back-compat branch with no row. "A record written before, holding only `review_input`, reads as that one path" is a branch of `generationRecord`'s read, and no planned row writes such a record and asserts the read.

8. **LOW** — Card 2, `TestRecords` row for an old record. The row repeats coverage that already exists.
   - Every existing `TestRecords` row reads a `judged.json` and scope record with no `files` or `loosened` key.
   - `TupleAddedToJudged` writes `{"tuples":[...]}` (`recordsguard_test.go:~100`) and reads it back through `ReadRecords`.
   - New omitempty fields cannot break that read. Either drop the row, or say what it pins beyond the existing rows.

9. **LOW** — Card 10, new top-level test. `TestRecipe_PRReviewReroutesToCodeBouncer` pins one routing value per recipe.
   - It sits beside `TestRecipe_WebsterReroutesToPlanReview` and `TestRecipe_FinalizeDeclaresNoReroute`, each a separate test of an `OnReroute` value (`loomrecipe/recipe_test.go:399`, `:424`).
   - Under "Test economy" (many items of one kind share one table), a table over the recipes' reroute edges would hold it instead of a third single-purpose test.

10. **LOW** — Card 9, `TestPRReview_Wait` row "a wake whose record carries the mark stops the session". The wait world (`prreview_wait_test.go:59-80`) sets no `fakePRSessions` that records stops. Without one, the assertion that the session was stopped cannot fail. The row needs a sessions fake that counts `StopStrand`, and the card should list it.

11. **LOW** — Card 3, halt ordering untested. "after any `does not match` halt the cells or cases give" orders two Stuck reasons, and no row combines a no-match cell with an uncaused file to pin which one wins.

**Tool calls made beyond my context**
- Read `verify_test.go` (`TestDriftVerify` start) and `recordsguard_test.go` (`TestRecords`), to see whether the drift fixture and the records test reach the planned rows.
- Read `newVerifyFixture`, `verifyFixture` and the judge-spawn subtests, to check whether the fixture has the seams and the fake judge for file rulings.
- Read the PR-Review fixtures and `TestPRReview_Call` / `TestPRReview_Session` heads, to check which helper fills `PullRequestDeps` and how dirty paths and sessions are scripted.
- Read the `prReviewTestDeps` callers, `TestRecipe_WebsterReroutesToPlanReview`, `TestLoadConfig_Loads` and `TestRecord_RoundTripAndUpdate`, to confirm the shared helper, the recipe test shape and the template-defaults map.
- Ran `lym quarry resolve` on the contracts/stencils test glyphs, `templateDefaults`, `prReviewTestDeps` and the verify fixture members, and grepped `TestBouncer_Amendment` for `ReviewInput`. This confirmed the unlisted helpers exist and how the amendment test uses the seam.
- Grepped `bouncer_amendment_test.go` and every test file referencing `ReviewInput`, to find the helpers card 7's type change forces.
