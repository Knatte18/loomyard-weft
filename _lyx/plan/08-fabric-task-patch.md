# Card 8 — fabric-task-patch

**Create:**
- `internal/fabricengine#TaskPatch`
- `internal/fabricengine#Fabric.ReadTaskPatch`
- `internal/fabricengine#Fabric.MergeInOnly`
- `internal/fabricengine#Fabric.WeftHeadSHA`
- `internal/fabricengine_test#TestFabric_ReadTaskPatch`
- `internal/fabricengine_test#TestFabric_MergeInOnly`

**Edit:**
- `internal/fabricengine_test#TestFabricHeadSHA_TracksWorktreeHead`
- `internal/fabricengine/doc.go`

**Intent:** The git reads a code segment's identity needs (D5), in `internal/fabricengine` per `PATTERN-fabric-git`, since `Fabric.Diff` returns changed paths only.

`ReadTaskPatch(base, skip)` reads the warp working tree's change against `base` (the told merge base): the full diff of tracked files, uncommitted changes included, plus every untracked file that is not ignored, each rendered as a new-file diff, minus the untracked paths `skip` names and minus paths under a `skip` entry ending in `/`.
A `skip` entry exempts a path only while it is untracked, so a tracked file of that name still counts.
`TaskPatch` holds `Head` (the commit HEAD stands at), `Full` (the full diff bytes in a stable order), `FullHash` (hex SHA-256 of `Full`) and `PatchID` (git's `patch-id --stable` of the zero-context form of the same diff, read through `gitexec.RunStdin`), all git through `gitexec`.
`MergeInOnly(since, parentTip)` reports whether every first-parent commit from HEAD back to `since` came from a Merge-In: a merge commit whose second parent is an ancestor of `parentTip`, or a commit that is itself an ancestor of `parentTip`, as a fast-forward Merge-In leaves it; it reads through `gitrepo` `Log` with `FirstParent` and `IsAncestor`.
`WeftHeadSHA` returns the commit the weft checkout's HEAD stands at, a one-line delegation to the records repo, so a caller reads the weft side without naming it.

`doc.go` lists the three reads.

`TestFabric_ReadTaskPatch` (integration, external package, `hubforge` pair): a committed task change and an uncommitted edit both appear; an untracked, non-ignored file appears and changes `FullHash`; an untracked skipped file does not; an untracked file under a `/`-ended skip entry does not; a tracked file of a skipped name does; a status-only change on the weft side leaves `FullHash` equal; a reordering-free rebase of the same change onto a moved base keeps `PatchID` while a task-side move of code changes `FullHash`.
`TestFabric_MergeInOnly`: after a merge of the parent into the task it reports true; after a fast-forward to a parent commit, true; after a task commit, false; after a merge commit whose second parent is not an ancestor of `parentTip`, false; with `since` equal to HEAD, true.
`TestFabricHeadSHA_TracksWorktreeHead` also asserts `WeftHeadSHA` tracks a new weft commit.

**ImpactSummary:** Additive read-only methods on Fabric; no existing caller changes.

**Commit:** 8: fabric: read the task's patch, its patch-id, merge-in-only history and the weft HEAD
