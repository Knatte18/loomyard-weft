# Finalize names the dirty parent checkout and its paths

Three batches ran, all `done`.

- Batch 01 (guard-error-carries-dirt, head `c17793ca7`): the dirty merge guard decides on a sorted path list, and `MergeGuardError` carries the checkout and the paths outside `Error()`.
  `Fabric.CodeDirtyPaths` and `Fabric.MidMerge` were added as the read-only parent reads.
  Deviation: `internal/fabricengine/mergelock_integration_test.go`.
- Batch 03 (finalize-dirty-reason, head `ae6feb82b`): Finalize's dirty-parent stuck reason is rendered per reason through `landingshed.DirtyParentRefusal`, the refusal spec was amended, and the halt note lost its `parent fabric status:` section and the parent read behind it.
  Deviations: `internal/haltnote/haltnote.go`, `internal/landingshed/finalize.go`.
- Batch 05 (finalize-goto-guard, head `050f7a6d9`): a goto onto Finalize is refused while the parent's code checkout is dirty, and a Go driver starting at a blocked stop runs the start check before its first step.
  No deviations.

No batch reported pattern departures, and no verb returned warnings.
