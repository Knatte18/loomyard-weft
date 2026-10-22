# Relevant files

- `internal/fabricengine/mergeerrors.go` `MergeGuardError`, `MergeGuardError.WorktreeDirty`, `IsWorktreeDirtyReason`, `newMergeGuardError` — the guard error that gains the paths field, and the helper D3 removes
- `internal/fabricengine/mergeguards.go` `pairDirtyReason` — the code-checkout-only dirty guard whose paths the error carries
- `internal/fabricengine/merge.go` `Merge`, `MergeIn`, `recheckMergePreconditionsUnderLock` — the three sites that call the dirty guard
- `internal/fabricengine/dirtiness.go` `worktreeDirty`, `trackedDirtyPaths` — the status probes the path list comes from
- `internal/fabricengine/diff.go` `Fabric.DirtyPaths` — the both-checkouts listing the halt note used, which showed the run record
- `internal/landingshed/finalize.go` `Finalize.commitParentRecords`, `Finalize.stuckOrCancelled`, `Finalize.mergeInStep` — Finalize's steps, the dirty arm D2 changes and the existing records commit
- `internal/loomcli/halt.go` `loomCLI.reflectHalt`, `loomCLI.parentDirtyPathsFromFabric`, `writeHaltNote` — the halt note writer and the parent read D3 removes and D4 reuses
- `internal/loomcli/cli.go` `loomCLI` — holds the `parentDirtyPaths` seam
- `internal/haltnote/haltnote.go` `Note`, `Render`, `Parse` — the halt note format losing its parent section
- `internal/loomcli/settledguard.go` `loomCLI.settledGuard`, `settledSegmentGuard` — the loom and darn goto guard D4 extends
- `internal/shedengine/goto.go` `GotoGuard` — the guard contract
- `internal/sheddriver/tackler.go` `TacklerCall.conclude`, `TacklerCall.resumePrecondition` — a refused resume goto hands back
- `internal/shedengine/producer.go` `ProducerDef` — `OnDone` identifies the rows that route into Finalize for D4
- `contracts/recipes/loom-recipe.yaml` — the Finalize row's engine and the row whose `on_done` routes to it
- `internal/sheddriver/driverules.go` `decideDrive` — a blocked stop goes to the tackler
