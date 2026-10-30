# Card 12 — retire-pr-review-keys

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine/template.yaml`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/darnengine#Config`
- `internal/darnengine#ConfigSpec`
- `internal/darnengine/template.yaml`
- `internal/darnengine#TestLoadConfig`
- `internal/darnengine/doc.go`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`

**Uses:**
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`

**Intent:**
D3: the stub's `pr_review` keys retire now that card 11's row reads neither.
`loomengine.Config` and `darnengine.Config` drop `PRReview`; both templates drop the `pr_review` line; each `ConfigSpec` drops the key from `Categories` and adds `"pr_review": configengine.CategoryNone` to `RetiredKeys`, so a hub file still holding `loom.pr_review` or `darn.pr_review` loads, the key is dropped from the stacked document, and `lym config reconcile` prunes it as it prunes every retired key.
`TestLoadConfig_Loads` gains the row `a retired pr_review in the repo default is ignored` beside the `fan_review` row, and darn's `TestLoadConfig` gains `a retired pr_review key still loads` beside the `tree_scope_verify` row.
`TestModules_EveryTemplateKeyClassified`'s `wantLocalKeys["darn"]` drops `pr_review`.
The darn doc's key list drops `pr_review`.

**ImpactSummary:** Two config fields with no remaining reader go; old hub files keep loading through the retired-key rule.

**Commit:** 12: retire loom.pr_review and darn.pr_review
