```yaml
lens: repo-invariants
outcome: reported
```

Lens: repo-invariants. For each card I checked test seams and tier tags, doc text the card falsifies, and card order.

**Per-card list**
- **Card 1** (fabric range reads)
  - Tier tag: held. The new tests are tier1, in the tier1 file beside `TestCommitLog_ListsTheRangeNewestFirst` (`internal/fabricengine/parenttips_integration_test.go:397`).
  - Doc text: held. Nothing it falsifies; `doc.go` is under Uses.
  - Order: held.
- **Card 2** (drift judgement types)
  - Seams: held. No `//lym:seam` marks `JudgedRecord`, `CheckResult` or `Judgement`, and `TestRecords` has no seam marker (`recordsguard_test.go:45-50`).
  - Doc text: held. `PATTERN-sole-parsers` names only the parser, so the new judgement keys falsify nothing.
  - Order: held.
- **Card 3** (Drift-Verify listing)
  - Seams: held. `driftengine#Deps` is seam-marked in envkit and loomcli tests, and envkit fills the two new seams.
  - Order: **broken**. `driftVerifyEntry`'s new required seams break `shedrecipe#TestDriftEntries` through the hand-built `driftTestEnv`; see F1.
  - Doc text: held. `driftengine/doc.go` is under Edit.
- **Card 4** (`pr_review_small_max` key)
  - Seams and doc text: held.
- **Card 5** (pull-request record fields)
  - Seams: held. No seam marker covers `pullrequest#Record`.
  - Doc text: held. `pullrequest/doc.go` describes no field the card changes.
- **Card 6** (plain includes in `ReadForEngine`)
  - Doc text: held. `stencilstore/doc.go` is under Edit, and the `PATTERN-stencil-ownership` background stays true.
  - Behavior of existing stencils: held. Plain includes appear only in seat stencils, which `ReadSet` reads. No production site fills an include map after `ReadForEngine`.
- **Card 7** (review inputs)
  - Seams: held.
    - The `Env.ReviewInput` rename covers the `nilLegal` key and `FullEnv`.
    - The shedadapters helpers `reentryInputRow` (`bouncer_reentry_test.go:170`) and `amendmentInputRow` (`bouncer_amendment_test.go:187`) are listed.
    - The loomcli tests that read `ReviewInput` are listed.
- **Card 8** (`pr-findings` verb)
  - Tier tag: held. `prfindings_test.go` is tier0. The test it copies, `TestPullRequestDeps_ParentReviewWantedReadsThePairsLayers`, is tier0 (`landingdeps_test.go:469-470`).
  - Order: held. It uses `prFindingsFile` (card 7), `Record` fields (card 5) and the config key (card 4), all from earlier cards.
- **Card 9** (send-back row)
  - Seams: held.
    - `landingshed#PullRequestDeps` is seam-marked in `landingdeps_test.go:46`, the loomcli wiring tests, `battencli/wire_seams_test.go:226` and `envkit_test.go`.
    - `pullRequestDeps` and envkit `LandingDeps` fill `ShedStatus`, and battencli builds no `PullRequestDeps` of its own.
    - The other hand-built `PullRequestDeps`, in `finalize_pr_test.go:489` and `publish_test.go:102`, do not go through `PRReview.checkWiring`.
  - Doc text: **broken**. The `PRReview` type doc is falsified and unlisted (F3). `landingshed/doc.go:435`, "Neither row has a reroute edge", is falsified but covered, since the card lists that `doc.go` under Edit.
  - Order relative to card 10: held. Before card 10 no mark can be written, because `sendBackEdge` answers false with no `OnReroute`.
- **Card 10** (recipes get `on_reroute`)
  - Tier tag: held. The new recipe test is tier0.
  - Doc text: **partly broken**. The recipe comments saying PR-Review's "Stuck is always a Halt" become false and the Intent does not name them (F2).
- **Card 11** (shared fragment)
  - Registration: held. `entries` and `TestStencils_TopLevelMarkers` are listed.
  - Order: held, since it comes after card 6.
- **Card 12** (wording): no Go code changes and no test, so held.

**Findings**

**F1 — BLOCKING, scope**
- **Location:** `_lym/plan/03-drift-verify-loosened-checks.md`, the seam list.
- **Summary:** Card 3 makes `driftVerifyEntry` require `Drift.ChangedFiles` and `Drift.CommitMessages`. The shedrecipe test env `driftTestEnv` builds `env.Drift` by hand without them. `requireDriftSeams` refuses on the first nil seam, so `TestDriftEntries`'s `VerifyBuilds` row, which expects a built `DriftVerify`, fails at card 3's gate. The card lists neither `internal/shedrecipe#driftTestEnv` nor `internal/shedrecipe#TestDriftEntries`.
- **Evidence:**
  - `internal/shedrecipe/entries_drift_test.go:20-50` (`driftTestEnv`, which fills neither seam).
  - `entries_drift_test.go:102` (`VerifyBuilds` row in `TestDriftEntries`, line 62).
  - `internal/shedrecipe/entries_drift.go:25-32` (`requireDriftSeams` returns the first nil seam).
- **Fix:** add `internal/shedrecipe#driftTestEnv` under card 3's `**Edit:**`, filling both seams with empty answers. Optionally add `internal/shedrecipe#TestDriftEntries` with rows for the two new nil seams.

**F2 — LOW, consistency**
- **Location:** `_lym/plan/10-recipes-pr-review-reroute.md`, the Intent.
- **Summary:** Once PR-Review reroutes, the recipe comments saying its "Stuck is always a Halt" are false. Card 10's Intent names only the "no edge out" header comments.
- **Evidence:**
  - `contracts/recipes/tapestry-recipe.yaml:522`
  - `contracts/recipes/sampler-recipe.yaml:240`
  - `contracts/recipes/darn-recipe.yaml:30` and the darn PR-Review row comment
- **Fix:** have card 10's Intent reword these lines too: PR-Review's Stuck is a Halt except a marked send-back's reroute.

**F3 — LOW, consistency**
- **Location:** `_lym/plan/09-pr-review-send-back-row.md`, the Edit list.
- **Summary:** The `PRReview` type's doc comment lists how the row ends: "The wait ends when the pull request is merged or approved, or closed or continued by the operator". Card 9 adds a reroute ending, which falsifies it, and the card does not list `internal/landingshed#PRReview`.
- **Evidence:** `internal/landingshed/prreview.go:29-45`.
- **Fix:** add `internal/landingshed#PRReview` under card 9's `**Edit:**` so its doc names the send-back reroute.

**Tool calls**
1. Seam markers on the touched structs, the `recordsguard_test.go` header, the tier of the ParentReviewWanted test, the `PullRequestDeps` construction sites, drift-deps test builders, and the shedrecipe, pullrequest and landingshed doc lines. My context lacked the seam markers and the other construction sites.
2. Battencli wiring, the shedrecipe drift test envs and the plain stencil includes. My context lacked the sites that build drift deps outside the manifest, and the include usage.
3. The bodies of `driftTestEnv` and `newTestEnv`, and the `TestDriftEntries` rows. Needed to confirm the missing seams.
4. The enclosing functions of the shedrecipe and shedadapters lines, and `requireDriftSeams`'s error shape. Needed to name glyphs and confirm the gate failure.
