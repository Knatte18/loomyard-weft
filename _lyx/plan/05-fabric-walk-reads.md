# Card 5 — Fabric reads a told-geometry config walk needs

**Create:**
- `plan:internal/fabricengine#Fabric.MergeBlocked` -> `func (f *Fabric) MergeBlocked() (bool, error)`
- `plan:internal/fabricengine#CodeWorktree` -> `type CodeWorktree struct`
- `plan:internal/fabricengine#CodeWorktrees` -> `func CodeWorktrees(worktreePath string) ([]CodeWorktree, error)`
- `plan:internal/fabricengine#OpenCodeWorktree` -> `func OpenCodeWorktree(path string) (*Fabric, error)`

**Edit:**
- `internal/fabricengine/midmerge_integration_test.go`
- `internal/fabricengine/openparent_integration_test.go`

**Uses:**
- `internal/fabricengine#Fabric.Commit`
- `internal/fabricengine#MidMerge`
- `internal/fabricengine#List`
- `internal/fabricengine#Open`

**Intent:**
Card 6's `internal/hubreconcile` is told geometry and never imports `internal/lyxcwd` (`PATTERN-told-geometry`), so it needs path-based fabric reads to enumerate a hub's code worktrees, open each one's fabric handle, and ask whether `Fabric.Commit` would refuse it as mid-merge (Decision 7).

`Fabric.MergeBlocked` (in `mergestate.go`) reports whether a fabric merge record exists or foreign git merge state sits on either side: exactly the two conditions `Fabric.Commit` refuses on (`*ErrMergeInProgress`, `*ErrForeignMergeState`), through the same `mergeRecordExists` and `foreignMergeStatePresent` probes, with no `MutationRecord`.
It is the one exported query for both conditions Decision 7 asks for; `MidMerge` is not reused because it takes a `*lyxcwd.Location` and also maps conflicted paths, which the walk does not need and which can fail on its own.
`Fabric.Commit` keeps its two separate refusals, since they carry different typed errors.

`CodeWorktree` (in a new `codeworktrees.go`) names one code worktree of a hub for a told-geometry caller: `Path` (the worktree root), `Anchor` (its anchor dir, where its `_lyx/config` lives) and `Main` (true for the hub's main worktree, the prime).
`CodeWorktrees` lists the code worktrees of the repository the worktree at `worktreePath` belongs to, through `List`, the main worktree first: a prunable entry is skipped, each remaining entry is resolved with `lyxcwd.ResolveWorktree` for its `Anchor`, and an entry whose directory no longer exists by the time it is resolved is skipped as removed; any other resolve failure is returned naming the path.
`OpenCodeWorktree` resolves the worktree root at `path` and opens it with `Open`; a `path` that does not exist returns an `*ErrMissingPath` naming it, so a caller can tell a pair removed since it was listed from a real failure.

Tests: `midmerge_integration_test.go`'s existing rows (clean, fabric-parked with and without conflicts, every foreign side and shape, and the unopenable pair) each also assert `MergeBlocked`: false on the clean pair, true on every parked and foreign row.
`openparent_integration_test.go` gains a test over a hubforge hub with two pairs: `CodeWorktrees` from either pair returns the prime first with `Main` true and both pairs with their anchors, on a root-anchored and a subpath-anchored hub; after one pair's directory is deleted by hand the listing omits it; `OpenCodeWorktree` opens a listed pair and returns `*ErrMissingPath` for the deleted one.

**ImpactSummary:** Additive: one read-only method and two read-only functions in fabricengine with no existing caller; `Fabric.Commit` is unchanged.

**Commit:** 5: fabric: list a hub's code worktrees by path, ask whether a commit would refuse as mid-merge
