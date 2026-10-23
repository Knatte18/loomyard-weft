# Card 48 — drop-publish-gate-kind

**Create:**
- `plan:internal/gateslot#MigrateConfig` -> `func MigrateConfig(existing []byte) ([]byte, []string, error)`
- `plan:internal/gateslot#TestMigrateConfig` -> `func TestMigrateConfig(t *testing.T)`

**Edit:**
- `internal/gateslot#goReadGates`
- `internal/gateslot#Config.TierList`
- `internal/gateslot#Config.GateCall`
- `internal/gateslot/template.yaml`
- `internal/gateslot#templateDefaults`
- `internal/gateslot#TestConfig_TierUse`
- `internal/configreg#Modules`
- `internal/loomcli#gateCallSource`
- `internal/loomcli#gateConfigWithTwoTiers`
- `internal/loomcli#TestLandingDeps_VerifyCommandReadsPlanAtCallTime`
- `internal/configcli#TestConfigOverRealHub`

**Extends:**
- `internal/gateslot/doc.go`

**Intent:** With Publish gone the `publish` gate kind has no site, so it goes from the gate schema (D8, D14); the `round` gate kind stays, since Webster's verify-gate fixer fork runs its call.
`goReadGates` drops `publish`, so the prompts' rendered gate lines no longer list it; `TierList` refuses an empty list for every gate, the `publish` exception gone, and `GateCall`'s empty-list branch and its doc go with it.
The template drops `gates.publish` and its comment names the gates lyx reads: card, batch, round, final and landing.
The gate module gains its migration, `gateslot.MigrateConfig`, which prunes `gates.publish` from a `gate.yaml` document and returns one line, `gates.publish: retired, removed`, keeping the `round` list and every other gate, and returns a document without it unchanged so a second pass is a no-op; a document that does not parse returns an error marked `configengine.ErrInvalid`.
`configreg` registers it as the gate module's `Migrate`, so `lyx config reconcile` prunes the key from the hub file, `hubreconcile.Ensure` runs it after the binary change, and a profile or `run_settings` document never carries it.
loomcli's gate call source's comment drops the empty-`publish` case, and its fixture and verify-command test drop the list.

`TestMigrateConfig` pins a document with a `publish` list pruned and its `round` list kept, one without it unchanged, and an unparseable document; `TestConfig_TierUse` drops the publish exception rows and gains a refusal of an empty list for every gate.
`TestConfigOverRealHub` gains the end-to-end row of D14, now that every migration stands: over a hub whose `loom.yaml` holds `webster_*` and `merge_*` perch keys, `alignment_pr_approve` and `rework`, whose `landing.yaml` holds `require_pr_to_base: []` and whose `gate.yaml` holds a `publish` list, `lyx config reconcile --apply` leaves the `code_*` keys, an empty `deploy_branches`, no retired key and the `round` list, and the hub's `profiles.yaml` byte-unchanged; a profile in it holding `webster_max_bounces` and `merge_max_bounces` then seeds a run whose loom layer reads `code_max_bounces`.

`doc.go` drops the `publish` gate.

**ImpactSummary:** One gate kind leaves the schema with a migration that prunes it; its readers are the gate config, the prompts' rendered lines and loomcli's call source, all listed.
