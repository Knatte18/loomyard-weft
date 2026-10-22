# Card 3 — finalize-dirty-reason

**Create:**
- `plan:internal/landingshed#maxListedDirtyPaths` -> `const maxListedDirtyPaths = 20`
- `plan:internal/landingshed#dirtyPathList` -> `func dirtyPathList(paths []string) string`
- `plan:internal/landingshed#DirtyParentRefusal` -> `func DirtyParentRefusal(checkout string, paths []string) string`
- `plan:internal/landingshed#Finalize.dirtyParentReason` -> `func (fz *Finalize) dirtyParentReason(parentHandle parentMerger, guardErr *fabricengine.MergeGuardError) string`
- `plan:internal/landingshed#recordingParentMerger.MidMerge` -> `func (m *recordingParentMerger) MidMerge() (fabricengine.MidMergeState, error)`

**Edit:**
- `internal/landingshed#parentMerger`
- `internal/landingshed#Finalize.Call`
- `internal/landingshed#recordingParentMerger`
- `internal/landingshed#TestFinalize_StuckReasons`
- `internal/landingshed_test#TestFinalize_OverRealHub`
- `internal/landingshed/doc.go`
- `contracts/specs/refusal-spec.md`

**Uses:**
- `internal/fabricengine#Fabric.MidMerge`
- `internal/fabricengine#MergeGuardError.AlreadyInProgress`
- `internal/fabricengine#MergeGuardError.NotSynced`
- `internal/fabricengine#MergeGuardError.DetachedHead`
- `internal/fabricengine#MergeGuardError.WorktreeDirty`
- `internal/landingshed#Finalize.stuckOrCancelled`
- `internal/landingshed_test#newFinalizeAt`

**Intent:**
D2's Finalize half, D1's acceptance criterion, and the refusal-spec amendment D2's texts rely on, in one commit.

`parentMerger` gains `MidMerge() (fabricengine.MidMergeState, error)`, which `*fabricengine.Fabric` satisfies from card 2, so Finalize reads the parked merge of the parent pair it already holds.

Rendering, one function for D2, D4 and D5 (`DirtyParentRefusal`, exported so `loomcli` reuses it):
- `dirtyPathList` renders each path Go-quoted (`strconv.Quote`), joined by `", "`, at most `maxListedDirtyPaths` of them; a longer list renders the first 20 and ends in `, and N more`.
  Quoting keeps a path holding a space or a comma one item, and the form holds no line break, so `stuckReason`'s `oneLine` leaves it intact.
- `DirtyParentRefusal(checkout, paths)` returns `the parent's code checkout <checkout> has uncommitted tracked paths: <list>; way forward: commit or discard them in <checkout> with plain git, then "lyx loom goto --to Finalize" and "lyx loom resume" in the task worktree`.

`Finalize.Call`'s steps 6-7 arm, for a guard error whose `WorktreeDirty()` is true, now returns `fz.stuckOrCancelled(ctx, fz.dirtyParentReason(parentHandle, guardErr))`; any other merge error keeps the generic `parent-side merge failed: %v` arm.
`dirtyParentReason` picks one form, in this order:
1. `AlreadyInProgress()`: the guard error's own text with no path list, then `; the parent's code checkout is <guardErr.Checkout>`, then, when `parentHandle.MidMerge()` returns `MidMergeParked`, `; the parked fabric merge is a <verb> of <source> in the parent pair` (premerge.go's naming), then the way forward:
   `way forward: the party that started that merge, or the orch or the operator when that party is gone, concludes it with "lyx fabric merge --continue" or discards it with "lyx fabric merge --abort" in the parent pair, then "lyx loom goto --to Finalize" and "lyx loom resume" in the task worktree; "--abort" hard-resets the parent's code checkout to the merge's start, so the operator first saves a copy of any change of their own in <checkout> outside it, since a plain git commit or stash there would take the staged merge with it; a session lyx refuses the verb from reports status: FAILED and the orch runs it`.
   A `MidMerge` error names no verb and source and keeps that way forward, logging the error through `logger.Warn`.
   A read reporting any kind but `MidMergeParked` (the record concluded or was removed after the guard) names no verb and source, and its way forward is only `"lyx loom goto --to Finalize" and "lyx loom resume" in the task worktree`, since `--continue` and `--abort` would refuse with no merge in progress.
2. `DetachedHead()`: the error's text, then `: the parent's code checkout <checkout> has uncommitted tracked paths: <list>; way forward: check out the parent branch "<ParentBranch>" in <checkout> with plain git, and when the checkout refuses to carry the changes, set them aside first with plain git and restore them on the branch after it; then commit them on the parent branch or discard them in <checkout> with plain git; then "lyx loom goto --to Finalize" and "lyx loom resume" in the task worktree`.
3. `NotSynced()`: the error's text, then the same subject, then `; way forward: commit them on the parent branch or discard them in <checkout> with plain git; then push or reconcile the parent branch "<ParentBranch>" in its own pair; then "lyx loom goto --to Finalize" and "lyx loom resume" in the task worktree` (Merge-In's diverged wording).
4. Otherwise, the dirt-only form, a source reason included: the error's text, `: `, then `DirtyParentRefusal(guardErr.Checkout, guardErr.DirtyPaths)`.
The two non-dirt parent reasons never come together (a detached HEAD reads as synced), so forms 2 and 3 are exclusive.
Form 2 is reached when the parent's code checkout is detached after step 3's `parentOpener` matched it by branch (`fabricengine.OpenParent` lists only a checkout on the parent branch) and before the merge's guard stage reads its HEAD; a checkout already detached at step 3 stops there with `no live pair for parent branch`, which this card leaves unchanged.
Its reaching test is the detached row of `TestFinalize_StuckReasons` below, whose fake merger returns the guard error that state produces.
Form 3 is reached when the guard's unfetched read of the local upstream ref shows a divergence at step 4 that step 1b3's origin reconcile did not stop, either because it arose between 1b3 and step 4 or because no reconcile is wired; its reaching test is the not-synced row of `TestFinalize_StuckReasons` below.
Every plain-git step names a checkout the message names, which the amended spec item below admits.

`contracts/specs/refusal-spec.md`'s `What a way forward is` item 3 widens from a plain `git` command in the task worktree to `a plain git command in a checkout the message names, when the work there is the operator's and no lyx verb covers it`.
`internal/landingshed/doc.go`'s Finalize passage says a dirty parent is Stuck naming the parent's code checkout and its tracked dirty paths, capped at 20, with the way forward per reason, and that the run records Finalize commits at step 2b never count as that dirt, since the guard reads the code checkout alone, whose `_lyx` link is excluded from its index.

Tests:
- `recordingParentMerger` gains `midMerge fabricengine.MidMergeState` and `midMergeErr error` fields, which `MidMerge` returns.
- `TestFinalize_StuckReasons` gains a `wantInOrder []string` field, asserted as substrings each found after the previous one, and a `midMerge`/`midMergeErr` pair copied onto the fake.
  Its guard-error rows build `&fabricengine.MergeGuardError{Reasons: …, Checkout: "/hub/parent", DirtyPaths: …}` with the reason literals, as the existing row does, and each asserts one merge call:
  the existing dirty row (paths `a b.go`, `c,d.go`) wants the error's text, `/hub/parent`, `"a b.go", "c,d.go"`, `commit or discard them in /hub/parent with plain git`, `lyx loom goto --to Finalize`, `lyx loom resume`, in that order;
  a 25-path row, fed already sorted as `codeDirtyPaths` hands them, wants the first 20 paths as given, quoted, and `and 5 more`, and not the 21st path;
  a 20-path row wants all 20 paths quoted and no `more`;
  a `source branch not found` plus `worktree dirty` row wants the error's text naming both reasons, the list and the dirt-only way forward, and no step naming the source branch;
  a `merge already in progress` plus `worktree dirty` row with a parked `merge` of `task-x` wants the error's text, `/hub/parent`, `merge of task-x`, `lyx fabric merge --continue`, `lyx fabric merge --abort`, `hard-resets the parent's code checkout`, `reports status: FAILED and the orch runs it`, the goto and the resume, and not `"a b.go"`;
  the same with `midMergeErr` set wants the same way forward and not `merge of`;
  the same with a `MidMergeNone` read wants the goto and the resume and not `merge --continue`, and the same with a `MidMergeForeign` read likewise;
  a `branch not synced to upstream` plus `worktree dirty` row wants the error's text, the list, `commit them on the parent branch or discard them`, `push or reconcile the parent branch`, the goto and the resume, in order;
  a `checkout is not on a branch` plus `worktree dirty` row wants `check out the parent branch`, `set them aside first`, `commit them on the parent branch or discard them`, the goto and the resume, in order.
- `TestFinalize_OverRealHub` gains a fourth step, after the third because its merge-in carries the earlier landings in cleanly: a fresh task pair with one commit, and in the parent's code checkout a committed `dirty.txt` modified and left uncommitted.
  A Finalize from `newFinalizeAt` is Stuck with a reason holding the parent's code checkout path as `filepath.EvalSymlinks(parentCode)` returns it, since the guard error's `Checkout` is the symlink-resolved path, and `"dirty.txt"`;
  after `git checkout -- dirty.txt` in the parent's code checkout a fresh Finalize is Done.
  This pins the dirt-only reason over a real guard read and the discard half of the operator-recovery criterion.
- D1's criterion, that dirt under `_lyx/` in the parent's records checkout lets the parent-side merge land, is the existing `TestMerge_DirtyTargetHalts`, whose weft half leaves a tracked `_lyx/tracked.txt` modified in the target's records checkout and asserts `Merge` lands: Finalize's step 4 is that `Merge` call on the parent pair.
- The commit half of the operator-recovery criterion is pinned link by link: card 5's `TestFinalizeDirtGuard` admits the goto once the code checkout is clean, `TestParentGuard_Moved`'s local-commit row reads a moved local tip as moved, and `TestGuardedRows_MovedParentReroutes` pins that Finalize turns a moved parent into a reroute Stuck before its parent-side merge.

**ImpactSummary:** Finalize's dirty-guard Stuck reason only, inside `landingshed`, plus one exported renderer `loomcli` adopts later; the generic merge-failure arm, Merge-In, Publish and every other surface of `MergeGuardError.Error()` keep their text, and the spec edit widens one way-forward item.
