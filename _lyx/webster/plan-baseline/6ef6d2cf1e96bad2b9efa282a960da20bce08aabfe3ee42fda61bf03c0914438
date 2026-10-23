# Card 23 — merge-review-retired

**Create:**
- `internal/yamlengine#DropKeys`
- `internal/yamlengine#TestDropKeys`
- `internal/loomengine#retiredLoomKeys`

**Delete:**
- `contracts/stencils/loom/loom-rubric-merge-review.md`
- `contracts/stencils#LoomRubricMergeReview`
- `internal/loomengine#PerchMerge`
- `internal/shedadapters#SubjectMerge`
- `internal/landingshed#MergeReviewNote`
- `internal/landingshed#noteList`
- `internal/landingshed#MergeRecord.SkipsReview`
- `internal/landingshed#TestMergeReviewNote`
- `internal/landingshed#TestMergeRecord_SkipsReview`
- `internal/loomcli#mergeSubject`
- `internal/loomcli#TestMergeSubject`
- `internal/loomcli#TestWire_MergeReviewSeams`

**Edit:**
- `contracts/stencils#entries`
- `contracts/stencils#rubricMarkerAllowlist`
- `contracts/stencils#TestLoomRubrics`
- `internal/shedadapters#RubricNotes`
- `internal/shedadapters#ReadRubric`
- `internal/shedadapters#noNote`
- `internal/shedadapters#BouncerConfig`
- `internal/shedadapters#Bouncer.rubricNotes`
- `internal/shedadapters#Subject.Unchanged`
- `internal/shedadapters#TestReadRubric`
- `internal/shedadapters#TestSubject_Unchanged`
- `internal/shedrecipe#Env`
- `internal/shedrecipe#bouncerEntry`
- `internal/shedrecipe#burlerReviewEntry`
- `internal/shedrecipe#withSubjectSeams`
- `internal/shedrecipe#TestBouncerEntry_ApproveSeam`
- `internal/shedrecipe#TestBouncerEntry_SkipSeam`
- `internal/shedrecipe#TestBouncerEntry_Subject`
- `internal/shedrecipe#TestBouncerEntry_BlockingEscalatesAndColorSegment`
- `internal/shedrecipe#TestBurlerHalfEntries_ColorSegment`
- `internal/shedrecipe#TestBurlerReviewEntry_RubricStencil`
- `internal/testkit/envkit#FullEnv`
- `internal/testkit/envkit#nilLegal`
- `internal/loomcli#loomCLI.wire`
- `internal/loomcli#loomPerches`
- `internal/loomcli#loomSegmentPerches`
- `internal/loomcli#loomSegmentSettings`
- `internal/loomcli#TestWire_DefaultConfig`
- `internal/loomcli#TestWire_ReviewKeysReachTheEnv`
- `internal/loomengine#Config`
- `internal/loomengine#Config.Caps`
- `internal/loomengine#reviewPerches`
- `internal/loomengine#ConfigOpenMaps`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#fanKeys`
- `internal/loomengine#segmentModelKeys`
- `internal/loomengine#ReviewSettings`
- `internal/loomengine#ResolveReview`
- `internal/loomengine#MigrateConfig`
- `internal/loomengine#retireSkipMergeReview`
- `internal/loomengine#copyWebsterModelsOntoMerge`
- `internal/loomengine/template.yaml`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestMigrateConfig_RetiredKeys`
- `internal/loomengine#TestResolveReview`
- `internal/loomengine#TestResolveReviewAndJudge_MalformedSpec`
- `internal/configsync#formerLoomValues`
- `internal/hubreconcile_test#loomFormerUnits`
- `internal/configreg#TestLookup`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/configreg#TestPrepareRunSettings`
- `internal/configreg#TestLayerRunSettings`
- `internal/configcli#TestConfigOverRealHub`
- `internal/configcli#TestDispatchSetProfiles`
- `internal/battencli#TestPrepareRunSettings`
- `internal/loomrecipe#TestShippedRecipes_FanSettingsAgreeWithinSegment`

**Extends:**
- `internal/yamlengine/doc.go`
- `internal/shedadapters/doc.go`

**Intent:** Remove the Merge-Review perch's machinery and keys (D6, D14), now that card 22 took its rows out of the recipe.
The `merge-overlap` skip seam, the `merge` approve seam and the `merge` subject go: `bouncerEntry` refuses those key values with its existing unknown-value error, `Env` drops `SkipMergeReview`, `ApproveMergeReview`, `MergeReview` and `MergeSubject`, `shedadapters.SubjectMerge` goes and `Subject.Unchanged` loses its merge case, and `wire` stops filling them.
`RubricNotes` drops `MergeReview` and `ReadRubric` stops filling a `merge_review` marker; the `loom-rubric-merge-review` stencil and its registration go, with the rubric marker allowlist entry.
`MergeRecord.SkipsReview` and `MergeReviewNote` go; the overlap computation stays (`computeOverlap`), since Finalize's large-merge judgement uses it (D6, D10).

`loom.yaml` loses `merge_reviewer_model`, `merge_fixer_model`, `merge_reviewer_fan`, `merge_fixer_fan`, `merge_max_bounces`, `merge_hard_max_bounces` and `merge_auto_accept`: `Config` drops their fields, `PerchMerge` goes from `reviewPerches` and every table built on it, and `ReviewSettings` drops `Merge` and `MergeKeys`.
`retireSkipMergeReview` and `copyWebsterModelsOntoMerge` keep running and name the merge keys by literal, so an old document still migrates through them first; `retireSkipMergeReview` still writes `merge_max_bounces: 0`, and the prune below runs after it (the plan writer's note).
`MigrateConfig` then drops each key `retiredLoomKeys` lists, through the new `yamlengine.DropKeys`, which removes each named top-level key with its comment and returns one line per removed key, `<key>: retired, removed`, leaving a document without them unchanged so a second pass is a no-op; the drop runs after the card-21 rename table, and later cards append `alignment_pr_approve` and `rework` to the list.
Since the profile check and run-settings layering run each module's `Migrate`, a profile or a board entry's `run_settings` holding `merge_max_bounces` still validates and seeds with the key dropped (D14; the Warn comes with the run-settings card).
`configsync`'s former loom values and the hub-reconcile test's former units drop the merge keys.

`TestDropKeys` pins a document holding two of the keys, one holding none, an empty document and one whose root is not a mapping; `TestMigrateConfig_RetiredKeys` gains rows dropping each merge key, one running `skip_merge_review: true` through to no merge key at all, and one showing a second pass unchanged.
The seam, subject, rubric and config tests drop their merge rows; `TestPrepareRunSettings` and `TestLayerRunSettings` gain a row whose `merge_max_bounces` is dropped rather than refused.

**ImpactSummary:** The Merge-Review perch's seams, rubric, perch constant and seven loom.yaml keys go, each reader listed; a migration drops the keys from every file, profile and run_settings document.
