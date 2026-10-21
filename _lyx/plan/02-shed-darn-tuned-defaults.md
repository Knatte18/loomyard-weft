# Card 2 — shed-darn-tuned-defaults

**Edit:**
- `internal/sheddriver/template.yaml`
- `internal/darnengine/template.yaml`
- `docs/overview.md`
- `internal/sheddriver#TestLoadConfig`
- `internal/darnengine#templateConfig`
- `internal/darnengine#TestLoadConfig`
- `internal/loomcli#TestWire_DarnRecipe`
- `internal/configreg#TestSettingLookup`
- `internal/battencli#TestChildDriverKind`

**Intent:**
D1's shed and darn values become the embedded templates' values.
`internal/sheddriver/template.yaml` sets `driver_kind: go`, its comment saying `go` is the template default and `llm` is selected by a hub, a module's own `driver_kind`, a task's run settings or the run's `--driver`.
`internal/darnengine/template.yaml` sets `darn_reviewer_model` and `darn_fixer_model` to `sonnet[medium]`, `darn_reviewer_fan: ["review-code", ""]` and `darn_fixer_fan: fix-code`, each comment rewritten to describe the new default instead of calling an empty value the default.

What it lets through, per D1: Darn-Review's reviewer and fixer drop to `sonnet[medium]`, bounded by the keys being hub-pinnable with `lyx config darn --layer default --set`; and no run uses the llm driver unless one of the selectors above picks `llm`.

Tests whose fixtures rewrite the old template text by string replacement would silently stop rewriting, so each follows the new text:
- `sheddriver`'s `TestLoadConfig` `templateDefaults` takes `DriverKind: "go"`, and its rows that expect the template's driver kind follow.
- `darnengine`'s `templateConfig` takes the new reviewer and fixer models and fans; `TestLoadConfig`'s rows that `strings.Replace` `darn_reviewer_model: sonnet[high]`, `darn_fixer_model: ""`, `darn_reviewer_fan: ""` or `darn_fixer_fan: ""` replace the new lines instead, keeping each row's own rewritten value.
- `TestWire_DarnRecipe`'s "the template's empty fans and models" row expects the template's new review and fix models and fan settings (the review key and the fix key are then both darn's own), and its "set fans and models" row and the unregistered-model row replace the new template lines.
- `TestSettingLookup`'s hub-wide row sets `darn_fixer_fan` to a value other than the new template's `fix-code`, so it still proves the board file answers rather than the template.
- `TestChildDriverKind`'s row named for shed.yaml's value seeds `driver_kind: llm` over the new `go` template line and expects `llm`, so the row still proves shed.yaml decides when both overrides are empty.

Every run armed or seeded without a typed driver now gets `go`, so the implementer greps the test files for `driver_kind`, `DriverKind` and `llm` as an expected seeded driver, and fixes in this card's commit any assertion that reads the template's driver kind rather than seeding its own; the files to check first are `internal/battencli`'s `arm_seed_test.go`, `runsettings_test.go` and `lifecycle_integration_test.go`, `internal/loomcli/sharedbootstrap_test.go` and `internal/shedbuild`'s `routing_test.go` and `parse_test.go`.
`docs/overview.md`'s child-driver sentence, which says the fallback ends at `shed.yaml`'s `driver_kind`, "`llm` by default", says `go` by default.

**ImpactSummary:** two templates' defaults and comments; five test functions whose fixtures replaced or mirrored the old template text follow the new text, no production code changes.
