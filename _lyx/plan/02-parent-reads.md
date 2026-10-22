# Card 2 — parent-reads

**Create:**
- `plan:internal/fabricengine#Fabric.CodeDirtyPaths` -> `func (f *Fabric) CodeDirtyPaths() (checkout string, paths []string, err error)`
- `plan:internal/fabricengine#Fabric.MidMerge` -> `func (f *Fabric) MidMerge() (MidMergeState, error)`

**Edit:**
- `internal/fabricengine_test#TestStatus_MergesUncommittedChangesBothSides_ExcludesWeftArtifacts`
- `internal/fabricengine_test#TestMidMerge_FabricParkedWithConflicts_Parked`
- `internal/fabricengine/doc.go`

**Uses:**
- `plan:internal/fabricengine#codeDirtyPaths`
- `internal/fabricengine#MidMerge`
- `internal/fabricengine#Fabric.DirtyPaths`

**Intent:**
The two read-only parent reads the later cards need, both on an opened `*Fabric`, so neither caller rebuilds a location or runs git itself (`PATTERN-fabric-git`).

`Fabric.CodeDirtyPaths` returns the code checkout's absolute path (`f.warpPath`) and `codeDirtyPaths(f)`: the same read the dirty merge guard decides on (D4), worktree-relative and sorted, never the records checkout, which `Fabric.DirtyPaths` also lists.
Its godoc says so, and names `Fabric.DirtyPaths` as the both-checkouts listing it is not.
A read failure is returned wrapped as `fabricengine: code checkout dirty paths: %w`.

`Fabric.MidMerge` resolves the pair's location from its code checkout with `lyxcwd.ResolveWorktree(f.warpPath)`, as `Fabric.MergeIn` already does, and returns `MidMerge` over it, so Finalize reads the parked merge of the parent pair it holds (D2's note: on the parent pair's location, never the task worktree's).
A resolve failure is returned wrapped; it carries no `MutationRecord`, as `MidMerge` does not.

`internal/fabricengine/doc.go` names both reads beside `MidMerge(l)` among the package's read-only probes.

Tests:
- `TestStatus_MergesUncommittedChangesBothSides_ExcludesWeftArtifacts`, whose fixture dirties a tracked `README` and an untracked file on the code side and a tracked config file on the records side, asserts `CodeDirtyPaths()` returns the code checkout, compared after `filepath.EvalSymlinks` on both sides since a temp root may be a symlink, and exactly `[]string{"README"}`.
- `TestMidMerge_FabricParkedWithConflicts_Parked` asserts `f.MidMerge()` equals `MidMerge(h.Location)` field for field on the parked pair.

**ImpactSummary:** two additive read-only methods on `Fabric` with no caller yet; the other edits are assertions added to two existing integration tests and one doc line.
