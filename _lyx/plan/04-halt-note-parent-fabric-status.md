# Card 4 — halt-note-parent-fabric-status

**Create:**
- `plan:internal/fabricengine#IsWorktreeDirtyReason` -> `func IsWorktreeDirtyReason(text string) bool`
- `plan:internal/fabricengine#Fabric.DirtyPaths` -> `func (f *Fabric) DirtyPaths() ([]string, error)`
- `plan:internal/loomcli#loomCLI.parentDirtyPathsFromFabric` -> `func (c *loomCLI) parentDirtyPathsFromFabric() ([]string, error)`
- `plan:internal/loomcli#renderParentFabricStatus` -> `func renderParentFabricStatus(paths []string, err error) string`

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
`Fabric.DirtyPaths` returns every uncommitted path of either side of the pair as an absolute path under its own checkout, in the order `Status` reports them, with no side label.
`loomCLI` gains the seam `parentDirtyPaths func() ([]string, error)`, which `newLoomCLI` sets to `parentDirtyPathsFromFabric`: it reads the recorded parent branch through `readRecordedParentBranch`, opens the parent pair through `fabricengine.OpenParent` and returns its `DirtyPaths`.
`haltNote` gains `ParentFabricStatus string`;
`reflectHalt` fills it through `renderParentFabricStatus` over the seam only when `fabricengine.IsWorktreeDirtyReason(n.Reason)` holds, and `writeHaltNote` appends a `parent fabric status:` section listing one path per line, the error text when the read failed, or nothing when the field is empty.
No stencil runs `lyx fabric status`; the reflection stencil gains one sentence: a halt note with a parent fabric status section attributes a dirty-tree halt to the tree the section names.
Tests: `halt_test.go` gains a row rendering the section from paths and from an error, and a case in the post-run halt test with a fake seam asserting a dirty-reason halt carries the section and another reason does not;
the merge vocabulary test gains an assertion that `IsWorktreeDirtyReason` matches a guard error's text and not another reason, and the status integration test gains a step asserting `DirtyPaths` returns both sides' paths as absolute paths under their checkouts.

**ImpactSummary:** Two fabricengine reads and one loomcli seam; the halt note gains an optional section and nothing else in the halt path changes.
