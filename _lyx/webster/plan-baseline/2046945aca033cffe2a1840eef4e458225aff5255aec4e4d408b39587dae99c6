# Card 4 — halt-note-parent-fabric-status

**Create:**
- `internal/fabricengine#IsWorktreeDirtyReason`
- `internal/fabricengine#Fabric.DirtyPaths`
- `internal/loomcli#loomCLI.parentDirtyPathsFromFabric`
- `internal/loomcli#renderParentFabricStatus`

**Edit:**
- `internal/loomcli#loomCLI`
- `internal/loomcli#newLoomCLI`
- `internal/loomcli#haltNote`
- `internal/loomcli#writeHaltNote`
- `internal/loomcli#loomCLI.reflectHalt`
- `internal/loomcli/halt_test.go`
- `internal/fabricengine/mergevocab_test.go`
- `internal/fabricengine/diff_integration_test.go`
- `contracts/stencils/friction/friction-template-reflection.md`
- `cmd/lyx/testdata/test-budget.yaml`

**Intent:** D11: a halt whose reason names a dirty worktree carries the parent pair's dirty paths in its halt note, so the reflection attributes the halt to the right tree.
`IsWorktreeDirtyReason` reports whether `text` contains the dirty-worktree guard reason, keeping the unexported constant inside fabricengine the way `MergeGuardError.WorktreeDirty` does;
`Fabric.DirtyPaths` returns the tracked uncommitted paths of both checkouts of the pair, through the tracked-scope porcelain read the dirty-worktree guard itself makes, so an untracked file the guard never objected to is never listed; each path is absolute under its own checkout, with no side label.
`loomCLI` gains the seam `parentDirtyPaths func() ([]string, error)`, which `newLoomCLI` sets to `parentDirtyPathsFromFabric`: it reads the recorded parent branch through `readRecordedParentBranch`, opens the parent pair through `fabricengine.OpenParent` and returns its `DirtyPaths`.
`haltNote` gains `ParentFabricStatus string`;
`reflectHalt` fills it through `renderParentFabricStatus` over the seam only when `fabricengine.IsWorktreeDirtyReason(n.Reason)` holds, and `writeHaltNote` appends a `parent fabric status:` section listing one path per line, the error text when the read failed, or nothing when the field is empty.
No stencil runs `lyx fabric status`; the reflection stencil gains one sentence: a halt note with a parent fabric status section attributes a dirty-tree halt to the tree the section names.
Tests: `halt_test.go` gains a row rendering the section from paths and from an error, and a case in the post-run halt test with a fake seam asserting a dirty-reason halt carries the section and another reason does not;
the merge vocabulary test gains an assertion that `IsWorktreeDirtyReason` matches a guard error's text and not another reason, and the status integration test gains a step asserting `DirtyPaths` returns both sides' modified tracked paths as absolute paths under their checkouts and omits an untracked file.

**ImpactSummary:** Two fabricengine reads and one loomcli seam; the halt note gains an optional section and nothing else in the halt path changes.
