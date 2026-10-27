# Card 11 — gate-adopt-key

**Edit:**
- `internal/gateslot#goReadGates`
- `internal/gateslot#MigrateConfig`
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#TestMigrateConfig`
- `internal/testkit/envkit#fixedGateLists`
- `internal/gatecli#TestGateTest_Scopes`
- `internal/gatecli/doc.go`

**Create:**
- `plan:internal/gateslot#seedAdoptGate` -> `func seedAdoptGate(gates *yaml.Node) []string`

**Uses:**
- `internal/gatecli#scopesConfigWith`

**Intent:**
Decision 9: the adopt check runs under a gate key of its own, `adopt`.

- `template.yaml` gains `adopt: {tiers: [0], scope: all}` after `finalize`, and its comment naming the keys lym reads lists `adopt` among the scope-`all` keys.
- `goReadGates` gains `adopt` after `finalize`, so `Config.TiersValue` renders its line in the prompts that take the tiers.
- `seedAdoptGate` writes `adopt` into the gates mapping with a copy of `finalize`'s value, as a flow mapping right after it, when the mapping holds `finalize` and no `adopt`, and returns the line `gates.adopt: added with the value of gates.finalize`; a mapping without `finalize`, or already holding `adopt`, is left as it is. `MigrateConfig` calls it after the list migrations and the rename, so a `finalize` that a retired `landing` list just became is copied too, and its doc comment says so. A second pass changes nothing, since the file then holds `adopt`.
  A hub file holds `finalize` only when the hub tuned it (PATTERN-sparse-hub-base), so the copy is a tuned unit, never a template fill, and it lands through the reconcile alone (PATTERN-config-layers).
- `doc.go` names `adopt` among the keys lym reads, the migration's new step, and a Tier plan row `lym adopt check  adopt  all`.
- `fixedGateLists` gains `adopt: {0, 1}`.
- `internal/gatecli/doc.go`'s example `gates:` block gains an `adopt` line.

Tests:
- `templateDefaults` gains `adopt: {Tiers: [0], Scope: all}`, so `TestLoadConfig` and `TestTemplateConfigAndLimits` read the new template key.
- `TestMigrateConfig` gains rows: a file tuning `finalize` to `[0, 1]` and holding no `adopt` gains `adopt: {tiers: [0, 1], scope: all}` and the added line, and a second pass over the result returns it unchanged with no line; a file holding both keeps its `adopt`; a file with no `finalize` gains nothing. Every existing row whose input holds, or migrates into, a `finalize` and no `adopt` gains the `adopt` entry and line in its expected output.
- `TestGateTest_Scopes`' subtest `--gate with scope all runs every tier whole, ignoring --card ids` runs over each of `finalize` and `adopt` in turn, editing a file before each so neither carries, and asserts for each key the tiers run whole and the run record's scope form `all` and gate key; the `adopt` turn is the acceptance row for `lym gate test --gate adopt`, with no subtest of its own.

**ImpactSummary:** One template key, one Go-read key and one reconcile step; existing hub files gain `adopt` only where they tune `finalize`, and only the migration and scope tests change.

**Commit:** 11: Add the adopt gate key and seed it from a tuned finalize
