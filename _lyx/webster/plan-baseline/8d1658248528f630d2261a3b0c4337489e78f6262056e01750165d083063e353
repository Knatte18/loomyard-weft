# Card 4 — fabric: stage tracked edits and list untracked files mid-merge

**Edit:**
- `internal/gitrepo/merge.go`
- `internal/gitrepo/worktree.go`
- `internal/fabricengine/mergestage.go`
- `internal/fabricengine/mutation.go`

**Create:**
- `internal/gitrepo/stagetracked_integration_test.go`
- `internal/fabricengine/mergestagetracked_integration_test.go`

**Intent:**
Add the two merge-surface verbs card 5's `Resolve` needs (D10).

`gitrepo`:
`StageTrackedChanges()` runs `git add -u` through the checked gitexec call, staging every tracked modification and deletion and nothing untracked (no force-add, per the Never Force-Add Invariant);
`UntrackedFiles()` returns the repo-relative untracked, non-ignored paths through go-git's worktree status with `.git/info/exclude` honoured, the way `WorktreeChangedFiles` reads it.

`fabricengine`, in `mergestage.go` beside `MergeStageResolved`:
`MergeStageTracked() (StageResult, error)` stages tracked changes on the warp side of an in-progress fabric merge, refusing foreign merge state and a call with no merge record exactly as the guarded merge verbs do, and records a new `KindMergeTrackedStaged` mutation;
`MergeUntrackedFiles() ([]string, error)` returns the untracked, non-ignored paths in the task worktree, unified the way the conflict list is, so a caller never names a fabric side.
Junctioned `_lyx` and `.lyx` sit in `.git/info/exclude`, so they never show up as untracked.

Tests (integration tag, real repositories):
`StageTrackedChanges` stages a modified and a deleted tracked file and leaves an untracked one unstaged;
`UntrackedFiles` lists a new file and omits an excluded one;
in a conflicted fabric merge, a tracked non-conflicted edit staged by `MergeStageTracked` lands in the merge commit `MergeContinue` writes, and `MergeUntrackedFiles` names a file created during the merge.

**ImpactSummary:** two additive gitrepo methods and two additive fabricengine merge verbs with one new mutation kind; no existing caller changes.
