# Card 6 — loom reads lenses

**Edit:**
- `internal/loomengine#Config`
- `internal/loomengine#fanKeys` -> `func fanKeys(cfg Config) []lenses.FanKey`
- `internal/loomengine#LoadConfig`
- `internal/loomengine#twillLenses`
- `internal/loomengine#planLensDirective`
- `internal/loomengine#discussionLensDirective`
- `internal/loomengine#PlanTable`
- `internal/loomengine#DiscussionTable`
- `internal/loomengine/template.yaml`
- `internal/loomengine#lensNames` -> `func lensNames(resolved []lenses.Lens) []string`
- `internal/loomengine#templateConfig`
- `internal/loomengine#TestLoadConfig_Loads`
- `internal/loomengine#TestLoadConfig_Refuses`
- `internal/loomengine#TestResolveReview`
- `internal/loomengine#TestPlanTable`
- `internal/loomengine#TestDiscussionTable`
- `internal/loomengine#TestDiscussionTable_Refuses`
- `internal/loomengine#discussionTableDirs`
- `docs/overview.md`

**Delete:**
- `internal/loomengine#seedBurlerConfig`
- `internal/loomengine#malformedBurlerYAML`
- `internal/loomengine#writeBurlerYAML`

**Create:**
- `plan:internal/loomengine#seedLensesConfig` -> `func seedLensesConfig(t *testing.T, baseDir, contents string)`
- `plan:internal/loomengine#malformedLensesYAML` -> `const malformedLensesYAML = "fans: [unclosed\n"`
- `plan:internal/loomengine#writeLensesYAML` -> `func writeLensesYAML(t *testing.T, layout *lyxcwd.Location, content string)`

**Uses:**
- `plan:internal/lenses#FanList`
- `plan:internal/lenses#FanKey`
- `plan:internal/lenses#FanShapeError`
- `plan:internal/lenses#ValidateFanKeys`
- `plan:internal/lenses#Load`
- `plan:internal/lenses#ResolveFan`
- `plan:internal/lenses#Lens`
- `plan:internal/lenses#Config`

**Intent:** loom's fan keys, `discussion_lenses` and `plan_lenses` resolve through `lenses.yaml` (Scope: loom reads lenses and fans from `lenses`).
`Config`'s eight `*_fan` fields become `lenses.FanList`; `fanKeys` returns `[]lenses.FanKey`; `LoadConfig` maps a `*lenses.FanShapeError` onto its key and validates through `lenses.ValidateFanKeys`, so its refusals name `lenses.yaml`.
`twillLenses` loads through `lenses.Load` and resolves through `lenses.ResolveFan`; its refusals name `lenses.yaml`: `lenses.yaml unreadable for the %q fan ... fix lenses.yaml, or set the key to "" to turn the lens forks off` and `fan %q unresolvable through lenses.yaml ... define the fan and its lenses in lenses.yaml, or set the key to "" to turn the lens forks off`; its local slice is renamed so it does not shadow the package.
Every doc comment naming `burler.yaml` as the home of lenses or fans names `lenses.yaml` instead: those of `twillLenses`, `planLensDirective`, `discussionLensDirective`, `PlanTable`, `DiscussionTable`, `LoadConfig`, `Config`'s `DiscussionLenses` and `PlanLenses` fields, and the file header comments of `config.go` and `plan.go`, dropping "and the embedded template" where it described burler's per-name fallback.
`discussionTableDirs`'s doc comment names the twill fan's `lenses.yaml`, and `docs/overview.md`'s loom config paragraph names `lenses.yaml` for the fans the eight `*_reviewer_fan` and `*_fixer_fan` keys name and for those `discussion_lenses` and `plan_lenses` name, and its Discussion-Write lens-fork sentence ("with `discussion_lenses` naming a `burler.yaml` fan") and Plan-Write one ("With `plan_lenses` naming a `burler.yaml` fan") name a `lenses.yaml` fan, in this card since their behaviour changes here.
Every fan key comment and the `discussion_lenses` and `plan_lenses` comments in `template.yaml` say "fan name from lenses.yaml", the `discussion_reviewer_fan` one dropping its "(its fans fall back per name to the embedded template)" aside, since the load's fill now gives that.

Tests: `seedLensesConfig`, `malformedLensesYAML` and `writeLensesYAML` replace the burler-named helpers and write `lenses.yaml`; `TestLoadConfig_Loads` and `TestLoadConfig_Refuses` call them, and their expected refusal texts name `lenses.yaml` (`define the fan in lenses.yaml`, `fix lenses.yaml, or set the key to empty`) and the `lenses: ` prefix where they named `burler: `.
`TestPlanTable`, `TestDiscussionTable` and `TestDiscussionTable_Refuses` plant `lenses.yaml`, resolve their template fans with `lenses.ResolveFan` over `lenses.Load` of a temp dir holding no `_lyx/`, which degrades to the template, and their row names and expected texts name `lenses.yaml`; the repo-default row writes the board's `lenses.yaml`.
`lensNames` takes `[]lenses.Lens`, and `templateConfig` and `TestResolveReview` build `lenses.FanList` values.
`grep -rn 'burler.yaml\|burlerengine\.\(FanList\|FanKey\|FanShapeError\|ValidateFanKeys\|ResolveFan\|LoadConfig\|Lens\|Config\)' internal/loomengine` returns nothing after the card.

**ImpactSummary:** loomengine alone changes: the fan field type moves package with identical YAML behaviour, and the refusal and comment texts name lenses.yaml; the loom tests asserting the old texts are listed.
