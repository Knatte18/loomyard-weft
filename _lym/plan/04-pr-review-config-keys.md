# Card 4 — pr-review-config-keys

**Edit:**
- `internal/landingshed#Config`
- `internal/landingshed#ConfigSpec`
- `internal/landingshed#LoadConfig`
- `internal/landingshed/template.yaml`
- `internal/landingshed#TestLoadConfig_TemplateDefaults`
- `internal/landingshed#TestLoadConfig_InvalidKeys`
- `internal/configreg#TestModules_EveryTemplateKeyClassified`
- `internal/gateslot#goReadGates`
- `internal/gateslot/template.yaml`
- `internal/gateslot#templateDefaults`
- `internal/testkit/envkit#fixedGateLists`

**Extends:**
- `internal/landingshed/doc.go`
- `internal/gateslot/doc.go`

**Uses:**
- `internal/battencli#TestWire_EverySeamFilled`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/loomcli#TestWire_EverySeamFilled`

**Intent:**
D7 and D8: the two keys the PR-Review row reads.
landing.yaml gains `pr_reviewer: opus[high]  # model-spec for the PR-Review row's interactive session, the agent the operator discusses the pull request with (see contracts/specs/llm-model-spec.md)`, after `describe_timeout_min`.
`Config` gains `PRReviewer string` with tag `yaml:"pr_reviewer"`; `ConfigSpec` classifies it `configengine.CategoryNone`; `LoadConfig` validates it with `modelspec.Parse` like `conflict`, `describe` and `fixer`, refusing a bad spec as `landing config key "pr_reviewer": <parse error>`, and its doc comment names the key.
`TestLoadConfig_TemplateDefaults` asserts `PRReviewer` is `opus[high]`; `TestLoadConfig_InvalidKeys` gains a row with `pr_reviewer: not-a-spec[` refused naming `pr_reviewer`.
`TestModules_EveryTemplateKeyClassified`'s `wantLocalKeys["landing"]` gains `pr_reviewer`.
gate.yaml's template gains `pr-review: {tiers: [0], scope: diff}` after `drift-verify`, and its comment on the keys lym reads names `pr-review` among the diff-scoped ones; `goReadGates` appends `pr-review`; the gateslot test's `templateDefaults` gains the entry; envkit's `fixedGateLists` gains `"pr-review": {0, 1}` so a landing test's gate-tier lookup answers the key.
The landingshed doc's landing.yaml paragraph gains a sentence naming `pr_reviewer` as the PR-Review session's model-spec.
The gateslot doc's list of the keys lym reads names `pr-review` among the diff-scoped keys, and its row-to-key table gains the row `loom, darn, pawl  PR-Review  pr-review  diff`.
No reader of `PRReviewer` or of the `pr-review` key exists yet; cards 13 and 14 read them.

**ImpactSummary:** Two template additions with their tests and one test-kit table entry; every hub picks both defaults up from the template, and no current reader changes behavior.

**Commit:** 4: landing.yaml pr_reviewer and the pr-review gate key
