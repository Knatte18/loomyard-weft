# Relevant files

- `internal/loomengine/template.yaml` — the loom keys D1 nests, with comments that cross-reference keys
- `internal/loomengine/tapestry.yaml` — the plan and parts keys D1 nests
- `internal/loomengine/config.go` `ConfigSpec` — loom's categories, retired keys and the tapestry plan-key refusal
- `internal/loomengine/reinterpret.go` `FlagCodeAutoAccept` — an existing reinterpretation that reads a moved key
- `internal/configengine/layers.go` `Spec`, `ValidateLocal` — dotted categories and the retired-key refusal the move table derives from
- `internal/configreg/configreg.go` `Reinterpretation`, `Module` — the reinterpretation shape and module registration
- `internal/configreg/runsettings.go` `PrepareRunSettings`, `LayerRunSettings`, `refuseRetiredKeys` — run-settings and profile validation that refuses old names
- `internal/configsync/configsync.go` `ReconcileAll`, `Result` — the reconcile that applies the move table to every file layer
- `internal/hubreconcile/walk.go` `reconcileHubWide`, `reconcileWorktree` — the walk order: configsync before reinterpretations, worktree commits
- `internal/hubreconcile/reinterpret.go` `applyReinterpretations`, `carryBeamTaskSettings` — reinterpretation markers, orch mail, per-machine carry precedent
- `internal/dobbyengine/reinterpret.go` `ReinterpretPerTaskSettings`, `CarryBeamTaskSettings` — a comment-keeping hub file rewrite and a board-to-worktree carry
- `internal/runprofile/template.yaml` — profile blocks holding flat loom and tapestry keys
- `internal/driftengine/template.yaml` — drift keys that stay flat
- `internal/darnengine/config.go` `ConfigSpec` — retired keys with a way forward
- `pattern/PATTERN-config-layers.md` — the layers rule the move table extends
