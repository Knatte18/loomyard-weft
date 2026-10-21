# Relevant files

- `internal/gateslot/config.go` `ConfigSpec`, `LoadConfig`, `withConfigWayForward`, `decodeConfig` — gate's key classification and strict loader, where `slots` and `go_parallel` are reclassified
- `internal/gateslot/doc.go` — documents the gate config as hub-file only
- `internal/gateslot/pool.go` `Pool`, `Limits`, `Dir` — the slot pool lives under the board dir's `.lyx`, and reads limits at every acquire
- `internal/hubgeom/gateslots.go` `GateSlots` — the one constructor every gate site uses, loading through `fabricengine.ConfigDirs`
- `internal/fabricengine/junctionnames.go` `ConfigDirs` — sets `Dirs.Board` and `Dirs.Anchor` for a hub load
- `internal/configengine/layers.go` `Layer`, `Category`, `Spec`, `LocalFile`, `LayerFile`, `BaseLayer`, `LocalKeys`, `LocalDenied`, `coveringCategory` — the layer stack and categories the machine layer joins
- `internal/configengine/doc.go` — the layer and local-rule contract the machine layer extends
- `internal/configcli/configcli.go` `dispatch`, `editOne`, `realHubWriteGuard`, `runReconcile` — `--layer` handling, the prime-only write guard and the reconcile entry
- `internal/configcli/menu.go` `layersHolding` — the menu's per-module layer marks
- `internal/configcli/doc.go` — documents `--layer` values, the hub-write guard and reconcile
- `internal/configsync/configsync.go` `ReconcileAll` — reconciles sparse and local layers; must leave the machine file alone
- `internal/hubreconcile/hubreconcile.go` `Geometry`, `Ensure` — keeps its stamp and lock under the board dir's `.lyx/config/`, beside the machine file it must not touch
- `internal/configreg/configreg.go` — registers gate as hub-wide with its `CheckLoad`
- `pattern/PATTERN-config-layers.md` — the category list that gains `machine`
- `pattern/PATTERN-hub-wide-config.md` — the hub-wide module's file rules
- `pattern/PATTERN-durable-vs-ephemeral-state.md` — where never-tracked override files live
