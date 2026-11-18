```yaml
lens: affected-set
outcome: reported
```

Affected-set lens: for each output, key, seam or behaviour a card changes, I listed what reads or asserts it and which cards cover it.

**Per change: the set found, and the cards covering it**

- **Card 1, new fabric reads `CommittedFileDiffs` and `CommitMessages`.** No caller yet outside card 3's `driftDeps`. Covered by cards 1 and 3.
- **Card 2, judge request, judgement and record fields.**
  - Set: `ParseJudgement` and `NewJudgementGate` (judge.go:194, unchanged), `TestParseJudgement`, `TestRecords`, `drift-template-judge.md`, whose marker row in markers_test is unchanged.
  - Covered by card 2.
- **Card 3, new `Deps` seams that `driftVerifyEntry` requires.**
  - Set: `loomcli#driftDeps`, `envkit#driftDeps`/`nilLegal`, the `verify_test` fixture, `shedrecipe#driftTestEnv` (entries_drift_test.go:24) with `TestDriftEntries`, and `shedrecipe#newTestEnv` (fixture_test.go:299, which fills only a partial Drift).
  - Covered by card 3, except `driftTestEnv`. **Partial.**
- **Card 3, `renderVerdict` gains `Loosened` lines.** Set: `TestVerdict`; the PR body and `drift_verdict` render `Verdict` unchanged. Covered by card 3.
- **Card 4, the `pr_review_small_max` key.** Set: `Config`, `ConfigSpec`, `template.yaml`, `templateDefaults`, `TestLoadConfig_Loads`, docs/overview. Covered by card 4.
- **Card 5, the four new record fields.** Set: `TestRecord_RoundTripAndUpdate`; the omitempty readers are unaffected. Covered by card 5.
- **Card 6, `ReadForEngine` writes plain includes in place.**
  - Set: every `ReadForEngine` caller (websterengine render, weaverengine render, gatefixer prompt, darnengine spec), `RenderPRReviewPrompt`, `RenderSessionPrompt`, `TestReadForEngine`, stencilstore `doc.go`.
  - Covered by card 6.
- **Card 7, `Env.ReviewInput` becomes `ReviewInputs`, and the generation record gains `review_inputs`.**
  - Set: shedrecipe recipe.go:302-304 and entries_bouncer.go:269/322, envkit.go:145/555, wiring.go:1164-1166, wiring_test.go:1014-1016 (in `TestWire_ReviewKeysReachTheEnv`), wiring_darn_test.go:111-118, `BouncerConfig` (bouncer.go:220-225), amendment.go, settlerecord.go:115-121, shedadapters doc.go:293/306, bouncer_amendment_test.go (`withReviewInput`, `amendmentInputRow`, `TestBouncer_Amendment` at 224/275/307/394), bouncer_reentry_test.go:182 (`reentryInputRow`), and **contracts/specs/shed-recipe-spec.md:110**.
  - Covered by card 7, except the spec. **Partial.**
- **Card 7, the rubric's `review_input` marker.** Set: `ReadRubric`, `rubricMarkerAllowlist`, `TestLoomRubrics`, `burlerReviewEntry`, `Bouncer.rubricNotes`. Covered by card 7.
- **Card 8, the new verb `pr-findings`.** Set: `Command`, loomcli `doc.go`, docs/overview. No CLI help-pin list names the loom verbs: cmd/lym/ghguard_test.go:113 is a package-level entry only. Covered by card 8.
- **Card 9, the required `PullRequestDeps.ShedStatus`.**
  - Set: `checkWiring`, `loomcli#pullRequestDeps` and `pullRequestDepsGuardedBy` (landingdeps.go:367/369), `envkit#LandingDeps` (envkit.go:239), `prReviewTestDeps` (prreview_test.go:667), `TestLandingDeps_EveryFieldPopulated`.
  - finalize_pr_test.go:489 and publish_test.go:102 build `PullRequestDeps` for other producers and never reach `PRReview.checkWiring`.
  - Covered by card 9.
- **Card 10, PR-Review gains `on_reroute: Code-Bouncer` in tapestry, darn and sampler.**
  - Set: the three recipe yamls with their header comments, `TestRecipe_*`, **`loomrecipe#TestRouting_ProgressAt`** (progress_test.go:73 pins PR-Review's reroute as empty), darnrecipe_test.go:203 (does not check `OnReroute`), **internal/loomrecipe/doc.go:25** ("PR-Review's Stuck is always a Halt"), and landingshed `doc.go`.
  - Covered by cards 9 and 10, except the progress test and the loomrecipe `doc.go`. **Partial.**
- **Card 11, the plain include in ten briefs.** Every brief is read through `ReadSet` or `ReadForEngine`: landing-template-fixer and darn-template-write go through the seat or `ReadForEngine`, and PR-Review and sampler switch in card 6. Covered by cards 6 and 11.
- **Card 12, prose only.** No reader.

**Findings**

1. **BLOCKING, design.** Location: `_lym/plan/10-recipes-pr-review-reroute.md`.
   - Card 10 gives tapestry's PR-Review `on_reroute: Code-Bouncer`, but `internal/loomrecipe#TestRouting_ProgressAt` pins PR-Review's routing as {Finalize, "", ""} with an empty reroute. No card edits that test, so card 10's own gate fails.
   - Evidence: internal/loomrecipe/progress_test.go:64 and :73. The subtest at :60 covers the tapestry tail.
   - Fix: add `internal/loomrecipe#TestRouting_ProgressAt` under card 10's `**Edit:**`, with the PR-Review row's reroute set to Code-Bouncer.
2. **BLOCKING, scope.** Location: `_lym/plan/03-drift-verify-loosened-checks.md`.
   - Card 3 makes `driftVerifyEntry` require `Drift.ChangedFiles` and `Drift.CommitMessages`. The shedrecipe test kit `driftTestEnv`, which `TestDriftEntries` builds every Drift entry from, fills neither, and the card does not list it.
   - So `TestDriftEntries`'s "builds with the kit's Env" rows fail. Its nil-seam rows also gain no rows for the two new seams.
   - Evidence: internal/shedrecipe/entries_drift_test.go:20-60; entries_drift.go:157-190.
   - Fix: add `internal/shedrecipe#driftTestEnv` and `internal/shedrecipe#TestDriftEntries` under card 3's `**Edit:**`. The first fills both seams with empty answers; the second gains rows for a nil `ChangedFiles` and a nil `CommitMessages`.
3. **MEDIUM, scope.** Location: `_lym/plan/07-bouncer-review-inputs.md`.
   - Card 7 renames `Env.ReviewInput` to `ReviewInputs` and makes it a list with a full-generation flag. contracts/specs/shed-recipe-spec.md:110 still says "Every kind also tells the Bouncer `Env.ReviewInput`, which stays optional", and no card lists the spec.
   - Evidence: contracts/specs/shed-recipe-spec.md:110.
   - Fix: add `//contracts/specs/shed-recipe-spec.md` under card 7's `**Edit:**`, naming `Env.ReviewInputs`.
4. **LOW, consistency.** Location: `_lym/plan/10-recipes-pr-review-reroute.md`.
   - internal/loomrecipe/doc.go:25 says "PR-Review's Stuck is always a Halt, so it carries no on_stuck". After cards 9 and 10, PR-Review returns a Stuck with Reroute and no Halt, and it declares `on_reroute`. Neither card lists this `doc.go`. Card 10 changes no non-test Go in loomrecipe, so `doc-go-unlisted` does not catch it.
   - Evidence: internal/loomrecipe/doc.go:25.
   - Fix: add `//internal/loomrecipe/doc.go` under card 10's `**Edit:**`, saying PR-Review's Stuck is a Halt or a send-back reroute to Code-Bouncer.

**Tool calls**
1. Searched for constructions of `PullRequestDeps` and `driftengine.Deps`, every `ReviewInput` reference, and the loomrecipe routing tests naming PR-Review: the manifest held only the definitions, not every constructor or asserting test.
2. Mapped the test functions that hold the `ReviewInput` references and the routing pins, and read the shedrecipe Drift test kits and shed-recipe-spec.md:105-115: these test helpers and the spec were outside the manifest.
3. Searched for tests that list loom verbs (pr-continue, sampler-done): to find any verb registry the new verb must join, which the manifest lacked.
4. Searched docs, specs and package docs for statements that PR-Review never reroutes: to find doc text card 10 falsifies, which the manifest lacked.
