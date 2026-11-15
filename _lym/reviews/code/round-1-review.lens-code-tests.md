```yaml
lens: code-tests
outcome: reported
```

**Task:** code-tests lens. For each behaviour the diff changes, the test that pins it, or none.

**Per behaviour**
1. `speed`, `light`, `solid` and `wide` darn blocks each carry `code_reviewer_model: "opus[high]"`: `TestTemplateLadder` (`runprofile_test.go:~203`) pins it. The new check asserts the key is present with exactly `"opus[high]"` for block `darn`, once per profile row.
2. The `docs` darn block carries the model beside the solo fan, sonnet writer and drift off: `TestTemplateDocsProfile` (`runprofile_test.go:~229`). Its full-map `DeepEqual` pins it.
3. The tapestry, pawl and sampler blocks stay without a model: `TestTemplateLadder` (`runprofile_test.go:~203`). The check fails when `present` is true for any block other than `darn`, and the `blockKeys` map now includes `sampler`.
4. The motif and arras blocks are unchanged: pinned by the existing `wantMotif` and `wantArras` `DeepEqual` checks in `TestTemplateLadder` (`runprofile_test.go:~186`). Nothing in the diff touches them.
5. The sampler block carries darn's rounds: `TestTemplateLadder` (`runprofile_test.go:~160`). The new `"sampler code"` entry compares `code_max_bounces` and `code_hard_max_bounces` against `perch(tt.darnCode)`.
6. The sampler block carries no solo fan: `TestTemplateLadder` (`runprofile_test.go:~199`). The sampler is in `blockKeys`, and no profile lists it in `solo`.
7. `TestPrepareRunSettings` (`runsettings_test.go:71`) pins that the default darn recipe writes `code_reviewer_model: opus[high]` into the loom doc, via `wantDocs`.
8. A file with no base picks up the new key: `TestReconcileHubWideAt_SeedOnlyFileFollowsItsTemplate`, row "with no base a lacking key is added…" (`configsync_test.go:1012`). Its `wantContains` entry ends at `code_reviewer_model: `, so it pins that the key is added next to the operator's kept `code_max_bounces: 4`.
9. A hub with an unedited earlier file picks up the new value, and the operator's own model value is kept: no row pins this for the model. See F1.
10. `TestConfigOverRealHub` and `TestEnsure_CarriesATemplateChangeIntoASeedOnlyConfigFile` only had their fixtures re-spelled with the model. Their older-template fixtures differ from the current template by the fan or round counts, never by the model.

**Findings**

- **F1: LOW, class `scope`.** `internal/configsync/configsync_test.go:947` (also `ensure_integration_test.go:1483` and `configcli_integration_test.go:~403`).
  - Every "older" template and base fixture already holds `code_reviewer_model: "opus[high]"` and differs only by `code_reviewer_fan` or the round counts.
  - No row has a base that lacks the model with a hub file equal to it, to show the unedited-earlier-file case picks the model up.
  - No row has a hub file with an operator's own, different `code_reviewer_model`, to show it is kept and listed under differs.
  - The generic key-level add and keep logic is covered by the sampler and fan rows (`configsync_test.go:985-1000`). So this is a coverage gap for the headline reconcile claim, not for new machinery.
  - Fix: either make the `older` fixture also drop the model, or add one row to the seed-only table. For example, `lightDarnOperator` with a different model such as `sonnet[high]`, and a base lacking the key, expecting the add or the differs entry.

- **F2: NIT, class `consistency`.** `internal/runprofile/runprofile_test.go:~160`.
  - The removed whole-map check (`sampler["loom"]` equals `darn["loom"]`) is replaced by a rounds-plus-fan check.
  - A stray extra key in a sampler block's `loom` would no longer fail.
  - The template carries none, so I judge this acceptable.
  - Optional fix: also assert that `len(samplerLoom)` matches the expected key count.

**Test economy.** `TestTemplateLadder` and `TestTemplateDocsProfile` are listed as contained, but nothing in the visible context shows `TestSettings`, `TestLoad` or `TestFormerRecipeNameRefusal` asserting the new model key. In `TestTemplateLadder`, the model assertions (item 3) pin which blocks carry the key, and the added `sampler` row (item 5) pins its rounds. `TestTemplateDocsProfile`'s full-map `DeepEqual` is the only full-map pin of the docs darn block.
- Not a redundancy finding: I cannot name a covering test that asserts the same thing.
- If the handler finds that `TestSettings` or `TestLoad` do assert it, the fix is to fold these into them or mark them `//testtiming:keep`.

**Tool calls**
1. Read `internal/configsync/configsync_test.go` lines 936-1065, because the diff showed only the changed lines and I needed the fixtures' older-template construction and the row set.
