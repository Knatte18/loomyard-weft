# Card 11 — gate-adopt-key

**Edit:**
- `internal/gateslot#goReadGates`
- `internal/gateslot/template.yaml`
- `internal/gateslot/doc.go`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#TestMigrateConfig`
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configsync#ReconcileHubWideAt`
- `internal/testkit/envkit#fixedGateLists`
- `internal/gatecli#TestGateTest_Scopes`
- `internal/gatecli/doc.go`

**Create:**
- `plan:internal/gateslot#seedAdoptGate` -> `func seedAdoptGate(gates *yaml.Node) []string`
- `plan:internal/gateslot#SeedAdoptGate` -> `func SeedAdoptGate(existing []byte) ([]byte, []string, error)`
- `plan:internal/gateslot#TestSeedAdoptGate` -> `func TestSeedAdoptGate(t *testing.T)`
- `plan:internal/configsync#TestReconcileHubWideAt_SeedsAdoptFromATunedFinalize` -> `func TestReconcileHubWideAt_SeedsAdoptFromATunedFinalize(t *testing.T)`

**Extends:**
- `pattern/PATTERN-test-speed.md`
- `docs/dotnet-hub.md`

**Uses:**
- `internal/gatecli#scopesConfigWith`
- `internal/gateslot#MigrateConfig`

**Intent:**
Decision 9: the adopt check runs under a gate key of its own, `adopt`.
The parent's decision "Adopt seed only for the hub gate.yaml" scopes its seed to the hub's base `gate.yaml`, and the parent's decision "A hub gate.yaml seeded from the prime or legacy files gets adopt in the same pass" has the seeding pass write it too.

- `template.yaml` gains `adopt: {tiers: [0], scope: all}` after `finalize`, and its comment naming the keys lym reads lists `adopt` among the scope-`all` keys.
- `goReadGates` gains `adopt` after `finalize`, so `Config.TiersValue` renders its line in the prompts that take the tiers.
- `seedAdoptGate` writes `adopt` into the gates mapping with a copy of `finalize`'s value, as a flow mapping right after it, when the mapping holds `finalize` and no `adopt`, and returns the line `gates.adopt: added with the value of gates.finalize`; a mapping without `finalize`, or already holding `adopt`, is left as it is.
  `SeedAdoptGate` runs it over a gate document's bytes, parsing and encoding as `MigrateConfig` does, and returns the bytes unchanged with no line when nothing is added. A second pass changes nothing, since the file then holds `adopt`.
- `MigrateConfig` is unchanged and never seeds `adopt`, since it also runs over the local-layer gate file (`configsync.reconcileSparseFile`), over a seed (`configsync.sparseSeed`) and over every run_settings or profile gate object (`configreg.PrepareRunSettings`, which logs each line at Warn on every read); all of those stay unseeded.
- `configreg.Module` gains `MigrateHubBase func(existing []byte) ([]byte, []string, error)`, whose doc comment says it runs only over a module's hub base file, after `Migrate` on a present file and over the prime or legacy seed on an absent one, and adds what only that file may gain; like `Migrate` it leaves the registry fingerprint unchanged. `Modules`' gate row sets it to `gateslot.SeedAdoptGate`.
- `ReconcileHubWideAt`'s present-file branch runs `m.MigrateHubBase` right after `m.Migrate` and appends its lines to the result's `Migrated`, so a `finalize` that a retired `landing` list just became is copied too, and the prune, strip and template-equal drop then judge the result.
  Its absent-file branch runs `m.MigrateHubBase` over the bytes `sparseSeed` returns for the `SeedPrime` and `SeedLegacy` seeds, so a hub `gate.yaml` this pass seeds holds `adopt` at once:
  `hubreconcile.Ensure` is gated on the build stamp and the seeding pass leaves it current, so no later reconcile would run until some later build.
  Its doc comment says both. No other reconcile path runs it.
  A hub file holds `finalize` only when the hub tuned it (PATTERN-sparse-hub-base), and `sparseSeed` strips a template-equal `finalize`, so the copy is a tuned unit, never a template fill, and it lands through the reconcile alone (PATTERN-config-layers).
- `doc.go` names `adopt` among the keys lym reads, the hub-base seed step and that only the hub base file takes it, and a Tier plan row `lym adopt check  adopt  all`.
- `pattern/PATTERN-test-speed.md`'s Gate keys table gains a row whose Key is `adopt`, whose Site is "`lym adopt check`, in-process through `gatecli.RunGate`", and whose Template scope is `all`.
- `docs/dotnet-hub.md`'s recommended `gates:` block gains `adopt: {tiers: [0], scope: all}` after `finalize`.
- `fixedGateLists` gains `adopt: {0, 1}`.
- `internal/gatecli/doc.go`'s example `gates:` block gains an `adopt` line.

Tests:
- `templateDefaults` gains `adopt: {Tiers: [0], Scope: all}`, so `TestLoadConfig` and `TestTemplateConfigAndLimits` read the new template key.
- `TestMigrateConfig` gains a row: a file tuning `finalize` to `[0, 1]` and holding no `adopt` is returned unchanged with no line, which pins that `MigrateConfig`, and so every caller but the hub base reconcile, never seeds `adopt`.
- `TestSeedAdoptGate` (tier0) rows: a file tuning `finalize` to `[0, 1]` and holding no `adopt` gains `adopt: {tiers: [0, 1], scope: all}` and the added line, and a second pass over the result returns it unchanged with no line; a file holding both keys keeps its `adopt`; a file with no `finalize` gains nothing.
- `TestReconcileHubWideAt_SeedsAdoptFromATunedFinalize` (tier0, a temp board directory): a hub `gate.yaml` holding a retired `landing` list `[0, 1]` and no `adopt`, reconciled with `Apply`, holds `finalize` and `adopt` both `{tiers: [0, 1], scope: all}`, and the gate result's `Migrated` holds the added line, and a second reconcile reports no `Migrated` line for gate and leaves the file byte-identical;
  a second row: an absent hub `gate.yaml`, seeded with `Apply` from a prime copy tuning `finalize` to `[0, 1]`, holds `finalize` and `adopt` both `{tiers: [0, 1], scope: all}` after that one reconcile.
- `TestGateTest_Scopes`' subtest `--gate with scope all runs every tier whole, ignoring --card ids` runs over each of `finalize` and `adopt` in turn, editing a file before each so neither carries, and asserts for each key the tiers run whole and the run record's scope form `all` and gate key; the `adopt` turn is the acceptance row for `lym gate test --gate adopt`, with no subtest of its own.

**ImpactSummary:** One template key, one Go-read key, one registry seam and one hub-base reconcile step; existing hub files gain `adopt` only where they tune `finalize`, local layers and run settings never do, and only the migration, seed and scope tests change.

**Commit:** 11: Add the adopt gate key and seed it from a tuned finalize in the hub gate file
