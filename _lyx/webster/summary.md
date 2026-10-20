# Lenses module: lenses and fans move out of burler

All six batches recorded `done`; none needed recovery.
The scope guard listed deviations (files outside a batch's own targets) for some batches, informational only; no batch reported a `Pattern-Departure:`.

- **01-yamlengine-per-name-maps** (head `3bc5aa97`): `internal/yamlengine` gained the per-name map kind in `Reconcile`, `Prune`, `SetValues` and `SetSparse`, and `configengine.Spec.PerNameMaps` is honoured by the unknown-keys warning and `Set`.
  Deviations: `internal/configengine/layers_test.go`, and `layers.go`, `layers_test.go`, `reconcile.go`, `reconcile_test.go`, `set.go`, `set_test.go` under `internal/yamlengine`.
- **03-lenses-module** (head `116e1a86`): `internal/lenses` created and registered in `configreg` (`Module.PerNameMaps`, covered by the fingerprint), with the template, `Load`, fan resolution and fan key grammar.
  Deviations: `internal/configcli/configcli.go`, `internal/configreg/configreg.go`, `internal/lenses/config.go`, `config_test.go`, `leaf_enforcement_test.go`.
- **04-configsync-per-name-reconcile** (head `e9937d85`): configsync reconcile adds per name and reports what the template lacks as `KeptExtra`, at every layer.
  Deviation: `internal/configsync/configsync_test.go`.
- **05-burler-reads-lenses** (head `478a0056`): `burlerengine`, `burlercli` and `loomcli` consume `internal/lenses`.
  No deviations.
- **06-loom-reads-lenses** (head `9ad15ad4`): loom's fan keys and twill lenses, and darn's fan keys, resolve through `lenses.yaml`.
  Deviation: `internal/loomengine/plan_test.go`.
- **08-retire-burler-module** (head `5d4f53d5`): burler's config deleted and deregistered, and `burler.yaml` migrates into `lenses.yaml` through two key moves, with `KeepTarget` added to `KeyMove`.
  Deviations: `internal/configreg/runsettings.go`, `internal/hubreconcile/ensure_integration_test.go`.

The plan-level verify runs at the gate after this turn.
