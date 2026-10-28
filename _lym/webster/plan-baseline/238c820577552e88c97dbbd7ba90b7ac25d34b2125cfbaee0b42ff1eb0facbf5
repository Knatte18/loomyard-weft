# Card 2 — solo-code-review-profiles

**Edit:**
- `internal/runprofile/template.yaml`
- `internal/runprofile/doc.go`
- `internal/runprofile_test#TestTemplateLadder`
- `internal/runprofile_test#TestTemplateDocsProfile`
- `internal/battencli#TestPrepareRunSettings`
- `internal/configsync#TestReconcileAll_SparseLayers`

**Uses:**
- `internal/configreg#TestProfilesCheckLoad`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/shedrecipe#TestBurlerReviewEntry_ClusterFan`

**Intent:**
D6: in `internal/runprofile/template.yaml`, the `light` profile's `loom`, `darn` and `pawl` blocks, the `speed` profile's `loom` block and the `docs` profile's `darn` block each add `code_reviewer_fan: ""` to their `loom:` flow mapping.
`speed`'s `darn` and `pawl` blocks, which set `code_max_bounces: 0`, and every block of `solid` and `wide` set no fan key, so they take the hub's `code_reviewer_fan`.
This card runs after card 1: the reclassified key is what lets configreg's check of a profile block, `checkLocalDocument` through `configengine.ValidateLocal`, admit the new value.

D7: `internal/runprofile/doc.go`'s paragraph on the starting profiles states that `light`, `docs` and `speed`'s loom block run the Code-Review round solo, through `loom.code_reviewer_fan: ""`, while `solid`, `wide` and the rest take the hub's fan.
The sentence "No key is open to the local layer." is about the profiles module's own keys and stays.

Tests:
- `TestTemplateLadder`'s rows gain the recipes whose block carries the solo fan: `speed` its `loom` block, `light` all three, `solid` and `wide` none.
  The subtest asserts `code_reviewer_fan` equals `""` in each named block's `loom` keys and is absent from every other block.
- `TestTemplateDocsProfile`'s `want` gains `"code_reviewer_fan": ""` in its `loom` map.
- `internal/battencli#TestPrepareRunSettings`' row "no profiles file takes the template's default for the recipe", a darn task seeded from the template's default `light` darn block, changes its `wantDocs` to `"code_hard_max_bounces: 2\ncode_max_bounces: 1\ncode_reviewer_fan: \"\"\n"`.
  That row is the D6 seed criterion's first step: the pair's local loom document carries the empty fan.
  The rest of the chain is pinned by tests this card reads and leaves as they are: `TestLoadConfig_Loads`' "hub stack" row "a local code_reviewer_fan of \"\" is open to the local layer", which card 1 adds, loads that local document over the template's list as `lenses.FanList{""}`, and `TestBurlerReviewEntry_ClusterFan`'s row "EmptyStringSettingRecordsASoloRound" records a solo round for an empty fan setting.
- `TestReconcileAll_SparseLayers` gains the row "a local loom file keeps a solo code_reviewer_fan": module `loom`, layer `configengine.LayerLocal`, content `code_reviewer_fan: ""\n`, `wantFile` the same bytes and `wantResult: true`, pinning the record's constraint that configsync's former-template-value strip, which lists `code_reviewer_fan: ""`, never removes the seeded value from a pair's local file.
- `TestProfilesCheckLoad`'s row "the template's starting profiles pass" checks every template block through configreg; `internal/configreg` is outside this card's packages, so the batch and `webster` gates run it, and it fails there if card 1 had not opened the key.

A run already seeded keeps the block it was seeded with, and the hub's live `profiles.yaml` is seed-only, so the change reaches only a hub seeded after the landing.

**ImpactSummary:** five template blocks gain one key; two runprofile template tests, one battencli Seed-Child row and one configsync reconcile row change, no Go production code changes.
