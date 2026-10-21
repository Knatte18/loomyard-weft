# Card 7 — review-amend-key

**Edit:**
- `internal/reviewbudget#Categories`
- `internal/reviewbudget#Values`
- `internal/reviewbudget#CheckResolved`
- `internal/reviewbudget#CheckLocal`
- `internal/reviewbudget#TestCheckResolved`
- `internal/reviewbudget#TestCheckLocal`
- `internal/reviewbudget/doc.go`
- `internal/loomengine#Config`
- `internal/loomengine#LoadConfig`
- `internal/loomengine/template.yaml`
- `internal/darnengine#Config`
- `internal/darnengine#LoadConfig`
- `internal/darnengine/template.yaml`
- `internal/darnengine/doc.go`
- `internal/darnengine#TestLoadConfig`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/loomengine#templateConfig`
- `internal/darnengine#templateConfig`

**Create:**
- `internal/reviewbudget#RunAmendKey`

**Intent:** D7: one run-wide amendment allowance key, `review_amend_max_bounces`, in the review budget key family.

`Categories` returns it as `CategoryNone` for every perch list, so `loom.yaml` and `darn.yaml` both declare it; it is no guard key, under the rule that a budget of 0 is the off switch.
`Values` gains `RunAmend int`; `CheckResolved` refuses a resolved value below 0 with the same refusal shape as the soft key, `must not be negative, got N; set it to 0 to settle every amendment with no review round, or to a positive integer`, and the `Route` way forward of the layer the value comes from.
`CheckLocal` bounds it at a minimum of 0 with no null, so a `run_settings` value below 0, a non-integer or a null is refused through `configengine.ErrInvalid`.
`loomengine.Config` and `darnengine.Config` gain `ReviewAmendMaxBounces int` (`yaml:"review_amend_max_bounces"`), read through their `LoadConfig`, which pass it to `CheckResolved`.
Both templates declare `review_amend_max_bounces: 1` with a comment: the review rounds an amendment generation gets after a settled segment's subject changed, clamped to the perch's own budget; 0 settles every amendment approved with no review round; settable per run through run_settings.
`internal/darnengine/doc.go`'s darn.yaml key list names the key, its "The two budget keys come from `internal/reviewbudget`" sentence becomes the three keys, and its `LoadConfig` paragraph names the below-0 refusal beside `review_max_bounces`'s; the key list in `internal/loomengine/config.go`'s file header names it beside the other budget keys.

Tests: `TestModules_EveryTemplateKeyClassified` adds `review_amend_max_bounces` to darn's `wantLocalKeys` in its sorted place, after `driver_kind`, since a `CategoryNone` key joins darn's derived `LocalKeys`; `TestCheckResolved` adds a below-0 row naming the key and its route; `TestCheckLocal` adds rows for -1, `~` and a string refused, and 0 accepted; `TestLoadConfig` (darn) asserts the template default 1 loads; loom's `TestLoadConfig_Refuses` adds a `review_amend_max_bounces: -1` row refused naming the key, so loom's pass of the value to `CheckResolved` is pinned.
The loom config tests that check every yaml tag appears in the template cover loom's half.
Both packages' `templateConfig` helpers, the whole Config the template loads to, gain `ReviewAmendMaxBounces: 1`: loom's is the want every `TestLoadConfig_Loads` row compares with `reflect.DeepEqual`, and darn's is the base of every `TestLoadConfig` want.

**ImpactSummary:** Additive key in two module templates and the shared budget checks; existing configs take the template default.

**Commit:** 7: reviewbudget: review_amend_max_bounces, the amendment allowance key of loom and darn
