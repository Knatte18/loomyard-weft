# Card 3 — drift-verify-gate-key

**Edit:**
- `internal/gateslot#goReadGates`
- `internal/gateslot#templateDefaults`
- `internal/testkit/envkit#fixedGateLists`
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`

**Extends:**
- `pattern/PATTERN-test-speed.md`
- `docs/dotnet-hub.md`
- `internal/gatecli/doc.go`

**Intent:**
Add the gate key `drift-verify`, the test gate over `Drift-Verify`'s own code changes (Scope, D6).

`template.yaml`'s `gates` gains `drift-verify: {tiers: [0], scope: diff}` after `code-burler-fix`, and the comment's list of keys lyx reads names it with scope `diff`.
`gates` is a per-name map, so a hub file that sets other keys keeps the template's value for this one.
`goReadGates` gains `drift-verify` last, so `Config.TiersValue` renders it to the prompts.
`templateDefaults` in `config_test.go` gains the row `"drift-verify": {Tiers: []int{0}, Scope: "diff"}`, which the template-load test compares against.
`envkit`'s `fixedGateLists` gains `"drift-verify": {0, 1}`, so a recipe built on the kit resolves the row's verify gate without a hub `gate.yaml`, and its doc comment drops its count of the keys ("the eight gate keys" becomes "the gate keys"), which the new key would falsify.

`internal/gateslot/doc.go`'s key lists gain `drift-verify` with scope `diff`, and its tier plan table gains the line `loom, darn  Drift-Verify  drift-verify  diff`.
`pattern/PATTERN-test-speed.md`'s gate key table gains the row `` `drift-verify` | `Drift-Verify`'s test gate over its agent's fixes | `diff` ``.
`docs/dotnet-hub.md`'s example `gates` block gains the key.
`internal/gatecli/doc.go`'s package-doc example `gates` block gains the line `drift-verify: {tiers: [0, 1], scope: diff}`.

No new behavior beyond a template value: the existing template-load test and `TestRecipeEngines`-style kit consumers cover it through the edited rows.

**ImpactSummary:** One template key and its mirrors in two test tables and the docs listing the keys; no site reads it until card 14-drift-recipe-rows.
