# Card 4 — pr-review-small-max-setting

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#ConfigSpec`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#templateDefaults`
- `//internal/loomengine/template.yaml`

**Extends:**
- `//docs/overview.md`

**Uses:**
- `internal/loomengine#TestConfigTemplate_ContainsEveryConfigYAMLTag`

**Intent:** D5: the loom key `pr_review_small_max`, an integer, default 3.

- `Config` gains `PRReviewSmallMax int` with tag `pr_review_small_max`, documented as the most findings PR-Review fixes in place between two send-backs, or before the first, when none is BLOCKING.
- `template.yaml` gains `pr_review_small_max: 3` beside `pr_review_ask_parent`, with an inline comment saying so and that it is settable per run through `run_settings`.
- `ConfigSpec` classifies it `none`, so the local layer, and with it a task's `run_settings`, may set it (`PATTERN-config-layers`); a new key changes no existing key's meaning, so it ships no reinterpretation.
- `TestLoadConfig_Loads` gains a row overriding it to 5, and `templateDefaults` gains `pr_review_small_max: 3`, so its template-defaults subtest asserts the default; that the local layer reaches the value is asserted where the value is read, card 8's `TestPRReviewSmallMax_ReadsThePairsLayers`.
- `docs/overview.md` names the key beside `pr_review_ask_parent` in the PR-Review and review-budget passages.

**ImpactSummary:** One new loom key with its template default and classification; no existing key or reader changes.

**Commit:** `4: pr-review-small-max-setting`
