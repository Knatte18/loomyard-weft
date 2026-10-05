# Card 8 — fabric-reset-pair-warp

**Edit:**
- `internal/fabricengine#pathOwnershipKind`
- `internal/fabricengine#pathOwnership`
- `internal/fabricengine#resolvePathOwnership`
- `internal/fabricengine#pathDirtinessKind`
- `internal/fabricengine#pathDirtiness`
- `internal/fabricengine#checkPathDirtiness`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/destroy_test.go`

**Create:**
- `plan:internal/fabricengine#Fabric.ResetPairWarp` -> `func (f *Fabric) ResetPairWarp(rec *Mutations, sha, parentBranch string, ownPaths []string) error`
- `internal/fabricengine/resetpairwarp_integration_test.go`

**Uses:**
- `internal/fabricengine#resetHardTo`
- `internal/fabricengine#Fabric.ResetHard`
- `internal/fabricengine#resolvePairWarpBranch`
- `internal/fabricengine#worktreeDirty`

**Intent:**
D6's destructive step: a gated reset of a task pair's warp checkout, beside `Fabric.ResetHard` in `destroy.go` (PATTERN-fabric-destruction-chokepoint, PATTERN-fabric-git).
`ResetHard` refuses on any tracked dirt and accepts the prime checkout, so it cannot express the operator's guard.

`ResetPairWarp` builds one `pathRequest` and runs it through the existing `resetHardTo` executor, which appends the `worktree_reset` entry to `rec` only after the reset happened:
- container: the hub, `filepath.Dir(f.warpPath)`, as `ResetHard` derives it;
- target: `f.warpPath`;
- ownership: a new kind, the pair warp checkout: the target is a registered linked worktree of the repository, never its main checkout, its checked-out branch is not `parentBranch` (the `resolvePairWarpBranch` rule), and the pair's weft checkout has `WeftBranchName(<that branch>)` checked out, so the branch is the pair's own;
  a detached HEAD fails ownership;
- dirtiness: a new kind, tracked changes except a caller-supplied path set: `git status --porcelain --untracked-files=no` through `worktreeDirty`'s probe, and every dirty tracked path outside `ownPaths` (worktree-relative, slash-separated) refuses, the reason naming each such path;
- force: always false.
Untracked files are left alone, as `git reset --hard` leaves them; the weft is never touched.
The checks run in the pipeline's fixed order, containment, ownership, dirtiness, force, and a refusal is a `*destructiveRefusal` that records nothing.
Bound: the method moves only the pair's own warp branch, never the main checkout, the parent branch or the weft, and discards uncommitted tracked changes only on the paths the caller names.

Tests:
`destroy_test.go` covers the new dirtiness and ownership declarations' request-shape rules;
`resetpairwarp_integration_test.go` (hubforge hub with a task pair, PATTERN-hubforge-fixtures) proves:
a reset to an older commit discards later commits and dirt on an own path, keeps an untracked file, and records one `worktree_reset` entry;
a dirty tracked path outside `ownPaths` refuses on dirtiness, naming the path, with HEAD and the file unchanged;
the prime checkout, a pair checked out on the parent branch, a pair whose weft is on another branch, and a detached HEAD each refuse on ownership.
Document the new primitive's request in `internal/fabricengine/doc.go` and in `destroy.go`'s executor list.

**ImpactSummary:** one new gated executor and two new declaration kinds in destroy.go; existing requests and executors are unchanged.
