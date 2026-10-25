# Card 21 — arbiter-key-copy

**Edit:**
- `internal/configsync#ReconcileHubWideAt`
- `internal/configsync#Result`
- `internal/hubreconcile#logResult`
- `internal/configcli#reconcileEnvelope`
- `internal/configreg/doc.go`

**Create:**
- `internal/configreg#KeyCopy`
- `internal/configreg#KeyCopies`
- `internal/configsync#copyKeys`
- `internal/configsync#TestReconcile_CopiesLoomArbiterKeysIntoDarn`

**Intent:**
Keep darn's arbiter behaviour across the move, as a config migration through `hubreconcile`'s walk (D8, `PATTERN-config-layers`): darn read `loom.yaml`'s arbiter keys until now.
`KeyCopy` names a key copied, not moved, from a source module's file to a target module's file of the hub's base layer only; `KeyCopies` lists `loom.arbiter`, `loom.arbiter_timeout_min` and `loom.arbiter_max_spawns` into `darn` under the same names.
`copyKeys` reads the board dir's base `loom.yaml` and `darn.yaml` and, for each listed key the loom file sets and the darn file does not, places the value in `darn.yaml` with the darn template's line comment, keeping the loom key; a darn file that sets the key keeps its value, so the copy is idempotent and a second walk changes nothing.
Only the hub layer is read: a value set in a worktree layer, the local layer or a task's `run_settings.loom` now applies to loom alone, and darn reads `shed.yaml` or its own override there.
`ReconcileHubWideAt` runs `copyKeys` after its key moves and before the per-module reconcile, and with `apply` writes the darn file as a sparse base (header kept) in the board pass's one commit; `Result` gains `Copied []string`, reported per target file; the walk's `logResult` logs it beside the moved keys, and `lyx config reconcile`'s `reconcileEnvelope` reports it as `copied` beside `moved`.
`configreg/doc.go` states the copy beside the key moves.
`TestReconcile_CopiesLoomArbiterKeysIntoDarn` (tier0, temp dirs, parallel) rows: a base loom file setting `arbiter_max_spawns: 0` and `arbiter: sonnet[high]` copies both into a darn base with none; a darn base already setting `arbiter_max_spawns: 2` keeps it; a dry run reports the copy and writes nothing; a second applied run reports nothing; a worktree layer's `loom.yaml` setting `arbiter_max_spawns: 0` copies nothing into any `darn.yaml`.

**ImpactSummary:** One new copy step in the board pass of the reconcile, writing only darn's base file, with its report field logged by the walk and shown by `lyx config reconcile`; the declared key moves and every worktree pass are unchanged.
