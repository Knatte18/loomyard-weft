# Card 5 — reconcile-floor-migrate

**Create:**
- `internal/yamlengine#RaiseIntFloor`
- `internal/shuttleengine#MigrateConfig`
- `internal/orchengine#MigrateConfig`
- `internal/loomengine#MigrateConfig`
- `internal/yamlengine/floor_test.go`

**Edit:**
- `internal/yamlengine/doc.go`
- `internal/shuttleengine/doc.go`
- `internal/orchengine/doc.go`
- `internal/configreg#Module`
- `internal/configreg#Modules`
- `internal/configsync#TestReconcileAll`
- `internal/configcli#Command`
- `internal/configcli/doc.go`
- `internal/configcli#TestConfigReconcileInGitRepo`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/shuttleengine#pollFloor`
- `internal/orchengine#pollFloor`
- `internal/loomengine#stepIdleTimeoutFloorMin`
- `internal/batcher#MigrateConfig`
- `internal/configsync#ReconcileAll`
- `internal/yamlengine#envMarkerRe`

**Intent:** Decision D5.
`lyx config reconcile` keeps a stored value its loader raises to a floor, so every command warns.
Put the one YAML rewrite in `yamlengine`, which already owns comment-preserving node editing: `RaiseIntFloor(existing, key, floor)` parses `existing` as a `yaml.Node` document and looks up the top-level mapping key `key`.
It returns `existing` unchanged with an empty line when the key is absent, holds a `${env:...}` marker (`envMarkerRe`), or holds any other scalar that does not decode as an integer, such as `abc`; when the value is an integer at or above `floor` it is likewise unchanged.
When the value is an integer below `floor`, or a null (`~`, `null` or an empty value), it sets the scalar to `floor`, re-encodes with two-space indent so every other key, value and comment is carried through as `batcher.MigrateConfig` does, and returns the line `"<key>: raised from <old> to <floor>"`, with `null` as the old value for a null.
A document that does not parse, or whose root is not a mapping, returns an error; an empty document is unchanged.
Add a short paragraph to `internal/yamlengine/doc.go` naming it, and pin it in the new `floor_test.go` with one table-driven test over those shapes (below, at, above, absent, null, env marker, non-integer, comments and sibling keys preserved, unparseable document).
Each engine's `MigrateConfig` is a thin hook: shuttle calls `RaiseIntFloor(existing, "poll_interval_ms", int(pollFloor.Milliseconds()))`, orch the same for its `pollFloor`, and loom `RaiseIntFloor(existing, "step_idle_timeout_min", stepIdleTimeoutFloorMin)`; each returns the rewritten bytes and the line as a one-element slice (nil when empty), wrapping a parse error with `configengine.MarkInvalid` as batcher does, so the floor stays declared once per engine and a hook never errors on an odd value.
Add one sentence each to `internal/shuttleengine/doc.go` and `internal/orchengine/doc.go` saying `MigrateConfig` raises the module's below-floor key to its floor for reconcile; `loomengine` has no `doc.go`, so its hook's doc comment carries that alone.
Register the three hooks in `configreg.Modules` on the `loom`, `orch` and `shuttle` rows, and widen the `Module.Migrate` field comment from "the keys its module declares retired" to also cover a key raised to its module's floor.
`ReconcileAll` already runs `Migrate` before the template compare and reports `Migrated` on a dry run and an applied run; `floor_test.go` alone pins the value shapes, so `TestReconcileAll` gains only the wiring each hook adds: one below-floor row per module, a `shuttle.yaml` with `poll_interval_ms: 500`, an `orch.yaml` and a `loom.yaml` below their floors, each checking that a dry run reports one `Migrated` line and writes nothing and that `--apply` writes the key at the floor inside the template-shaped merge.
`TestConfigReconcileInGitRepo`'s dry-run step asserts the `loom` entry omits `migrated` because loom has no migration, which the new hook makes false; retarget that assertion to the `reed` entry, a module that still has no hook and that the same step already reports, with its message naming reed.
Reword the reconcile `Long` in `configcli.Command` and the matching sentence in `internal/configcli/doc.go` from "A module that retires keys" to a module that retires keys or floors a value.
Raise `internal/yamlengine`'s row in `cmd/lyx/testdata/test-budget.yaml` by one for `floor_test.go`'s test.
Bound: a hook only raises a value its own loader already raises at every load, never lowers one and never touches another key, so the effective configuration never changes.

**ImpactSummary:** Three registry rows gain a hook and `ReconcileAll`'s existing seam carries them; the only behavior change is a reconcile report line and, under `--apply`, a rewritten below-floor key in three per-worktree config files.
