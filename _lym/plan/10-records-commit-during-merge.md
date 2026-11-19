# Card 10 — records-commit-during-merge

**Create:**
- `plan:internal/fabricengine#commitRecordsAroundMerge` -> `func commitRecordsAroundMerge(rec *Mutations, worktree, anchorRel string, relPaths []string, msg string) (sha string, committed bool, err error)`
- `plan:internal/fabricengine#MidMergeRecordsUnsettled` -> `const MidMergeRecordsUnsettled MidMergeKind = 3`
- `plan:internal/fabricengine#mergeReasonRecordsUnsettled` -> `const mergeReasonRecordsUnsettled = "merge records step unfinished"`
- `plan:internal/fabricengine#MergeStateOperation` -> `func MergeStateOperation(l *lymcwd.Location) (operation string, err error)`
- `plan:internal/fabricengine_test#TestRecordsCommit_DuringMerge` -> `func TestRecordsCommit_DuringMerge(t *testing.T)`

**Edit:**
- `internal/fabricengine#commitRecordsPathsHeld`
- `internal/fabricengine#CommitEmptyWithTrailers`
- `internal/fabricengine#Fabric.commitWeftLocked`
- `internal/fabricengine#takeoverBranch.commitEntry`
- `internal/fabricengine#takeoverBranch.land`
- `internal/fabricengine#migrateWeftMarker`
- `internal/fabricengine#recordAdoptedWeftAnchor`
- `internal/fabricengine#Fabric.Commit`
- `internal/fabricengine#Fabric.MergeAbort`
- `internal/fabricengine#Fabric.MergeContinue`
- `internal/fabricengine#MidMerge`
- `internal/fabricengine#MergeStateActive`
- `internal/fabricengine#pinnedMergeReasons`
- `internal/fabricengine#TestMergeErrors_PinnedStrings`
- `internal/landingshed#Finalize.dirtyParentReason`
- `internal/landingshed#Finalize.parkedMergeWayForward`
- `internal/landingshed#preMerge.clear`
- `internal/loomcli#loomCLI.refuseOverUnfinishedMerge`
- `internal/loomcli#ownMergeInLeftover`
- `internal/fabricengine_test#TestMergeStateActive`
- `internal/landingshed#TestFinalize_StuckReasons`
- `internal/landingshed#TestPreMerge_ClearsOwnParkedMergeIn`
- `internal/loomcli#TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/landingshed/doc.go`
- `internal/loomcli/doc.go`
- `internal/fabricengine#mergeState`
- `internal/fabricengine#Fabric.resetMergeSides`
- `internal/fabricengine#RecordsPathsAt`
- `internal/fabricengine#oneRepositoryAt`
- `internal/fabricengine#retryHeldIndexLock`
- `plan:internal/fabricengine#Fabric.SetAfterMergeStartForTest`
- `internal/hubforge#NewOneRepoHub`
- `internal/hubforge#NewHub`
- `internal/hubforge#AddPairWith`

**Intent:** D4's records commit while the worktree is mid-merge, in a one-repository hub. A records commit never skips there.

Every records commit primitive, `commitRecordsPathsHeld`, `CommitEmptyWithTrailers` and `Fabric.commitWeftLocked`, and `takeoverBranch.commitEntry` route a one-repository worktree with `MERGE_HEAD` present, or with a fabric merge record whose `RecordsUnsettled` is true, to `commitRecordsAroundMerge`, under the write lock.
The primitives already hold it. In that state `takeoverBranch.land` takes the write lock before the push lock, the order `lockWeftForReplay` takes them, and holds it across `commitEntry`,
so no blocking acquire of the write lock runs under the push lock. The route, by the merge's state:
- A merge whose index holds every records path as HEAD does: it builds the commit in a temporary index (`GIT_INDEX_FILE` under `lymtemp`) seeded from HEAD with the records paths taken from the worktree,
  writes it with `commit-tree -p HEAD` and the message the caller built, moves HEAD onto it with `update-ref` leased on the old HEAD, stages those records paths in the worktree's index,
  and moves the fabric merge record's `WarpStart` onto it when one exists. Concluding the merge then carries the records, `MergeAbort` resets to the new commit, and its conclude-landed guard does not read it as a landed conclude; `git merge --abort` keeps it as well.
- A fabric merge stopped before step 3 (`RecordsUnsettled` true): the temporary index takes the worktree's content of each records path whose index entry equals HEAD's or that neither holds, and HEAD's version of every other records path (step 2 wrote those from the source).
  It moves HEAD and `WarpStart` the same way, then resets to the new commit through `Fabric.resetMergeSides`, the hard reset `MergeAbort` runs, and deletes the merge record, all under the one hold of the write lock.
- Any other merge state (a records path whose index entry differs from HEAD's in a merge fabric did not start, an index conflicted on a records path, a cherry-pick, revert or rebase) is refused with the operation named;
  way forward: conclude or abort it with plain git, then retry, as `ErrForeignMergeState` names today. Card 11's seam waits on it through `MergeStateOperation`.
The staging of the worktree's index and the `update-ref` run through `retryHeldIndexLock`, as the plain commit does.
`Fabric.Commit` in a one-repository worktree takes the route for records-only file lists instead of refusing on a fabric merge record or foreign merge state; a list holding code paths still refuses as today.
`migrateWeftMarker` and `recordAdoptedWeftAnchor` commit `.lym-anchor` at the worktree root, outside the anchored records paths the route takes,
so in a one-repository worktree with `MERGE_HEAD` present, a fabric merge record standing or an operation `MergeStateOperation` names, they commit no marker and refuse with the state named;
way forward: conclude or abort it, then run the verb again. `migrateWeftMarker` reports the refusal in its pair's `Error`, and `recordAdoptedWeftAnchor` returns it.

`Fabric.MergeAbort` in a one-repository worktree first commits the worktree's uncommitted records the same way, by the rule matching its state, then aborts as today.
`Fabric.MergeContinue` on a merge whose `RecordsUnsettled` is true refuses with the guard reason `mergeReasonRecordsUnsettled`, a member of the reason const block, naming the unfinished records step;
way forward: `lym fabric merge --abort`, then merge again. `pinnedMergeReasons` and `TestMergeErrors_PinnedStrings` gain the reason.
A two-repository record never sets the field, so a two-repository parked merge continues as today.
`MidMerge` reports such a merge as `MidMergeRecordsUnsettled`, declared as the next member of the `MidMergeKind` iota block after `MidMergeForeign`, with the record's verb and source as for `MidMergeParked`.
`ownMergeInLeftover` and `preMerge.clear` treat a `MidMergeRecordsUnsettled` merge-in of the recorded parent branch at a landing producer as the row's own leftover, aborted and redone as a `MidMergeParked` one is,
since the one-repository `MergeAbort` handles an unsettled merge (D4); D5 keeps that abort-and-redo as it is.
`loomCLI.refuseOverUnfinishedMerge` handles `MidMergeRecordsUnsettled` as it handles `MidMergeParked`, consulting `ownMergeInLeftover` before it refuses, so the row's own unsettled merge-in proceeds.
For another party's unsettled merge every reader of the kind names only `lym fabric merge --abort`:
`Finalize.dirtyParentReason` with `parkedMergeWayForward` in the parent pair; `preMerge.clear`, which leaves it to the abort rather than its foreign-merge `default` text;
and `loomCLI.refuseOverUnfinishedMerge`, which refuses naming the abort.

`MergeStateOperation(l)` names the git operation a records commit cannot pass: `"merge"` for `MERGE_HEAD` or a conflicted index (`ConflictedFiles`), `"cherry-pick"`, `"revert"` or `"rebase"` for `CHERRY_PICK_HEAD`, `REVERT_HEAD` and the `rebase-merge` or `rebase-apply` directories of the worktree's gitdir, and `""` when none blocks.
In a one-repository worktree a merge blocks only when its index holds a records path unlike HEAD's, or is conflicted on one, and no fabric merge record stands unsettled; in a two-repository worktree any weft merge state blocks, as today.
`MergeStateActive` returns `operation != ""`, so its callers keep their meaning and its existing rows, the conflicted squash without `MERGE_HEAD` included, keep passing.

Tests:
- `TestRecordsCommit_DuringMerge` (tier1, `NewOneRepoHub`, a parent and a task pair, but for its last row's `NewHub` hub) rows:
  `TransitionDuringACodeMergeCommitsAndPushesAndConcludeKeepsIt` (a plain `git merge --no-commit` with a code conflict; `CommitAnchoredPathsWithTrailers` lands, `PushPairAnchored` pushes it, concluding keeps the commit);
  `GitMergeAbortKeepsTheRecordsCommit`; `FabricMergeAbortKeepsTheRecordsCommit`;
  `SyncDuringAMergeCommitsRecordsOnly` (`Fabric.Commit` of a records path over a plain merge lands around it);
  `CodePathListDuringAMergeRefuses` (`Fabric.Commit` of a code path over the same merge refuses as today);
  `CherryPickRefusesNamingTheOperation` (a conflicted `git cherry-pick`: the records commit refuses naming `cherry-pick` and the plain-git way forward);
  `CrashBeforeStepThreeCommitsAbortsAndPushes` (a merge whose `SetAfterMergeStartForTest` hook returns an error, leaving it before step 3; a status commit then lands, the merge is gone, and the status file is in HEAD, the worktree and origin with no source records in any);
  `UnsettledMergeReadsRecordsUnsettled` (the same stopped merge: `MidMerge` reports `MidMergeRecordsUnsettled`);
  `FabricMergeAbortOfAnUnsettledMergeKeepsAReceivingEdit`;
  `ContinueOfAnUnsettledMergeRefuses` (the refusal carries `mergeReasonRecordsUnsettled` and names the abort);
  `TwoRepositoryParkedMergeContinues` (a `NewHub` hub: a fabric merge-in parked on a conflict reads `MidMergeParked`, and once the conflict is resolved `MergeContinue` concludes it).
- `TestMergeStateActive` (tier1) gains rows over a one-repository pair: `MergeWithRecordsAsHEADReportsFalse`, `ForeignMergeWithARecordsPathUnlikeHEADReportsTrue`, `CherryPickInProgressReportsTrue`, each also asserting `MergeStateOperation`'s name.
- `TestFinalize_StuckReasons` (tier0) gains the row `UnsettledParkedMergeNamesOnlyAbort`: `midMerge` reports `MidMergeRecordsUnsettled`; the reason names `lym fabric merge --abort` and not `--continue`.
- `TestPreMerge_ClearsOwnParkedMergeIn` (tier0) gains two rows whose mid-merge reads `MidMergeRecordsUnsettled`:
  the row's own merge-in of the parent branch is aborted, and another source's unsettled merge gives a reason naming `lym fabric merge --abort` and not plain git.
- `TestRunDriverSpawnAndWait_MidMerge_SpawnRefusals` (tier0) gains two rows whose mid-merge reads `MidMergeRecordsUnsettled`:
  the row's own unsettled merge-in at Finalize proceeds, and another's refuses naming `lym fabric merge --abort`.

**ImpactSummary:** Records commits, the takeover entry's commit, MergeAbort, MergeContinue and MidMerge gain one-repository mid-merge branches, and the two marker commits refuse mid-merge there; MergeStateActive keeps its signature and meaning for its three outside callers; Finalize gains one way-forward case, and the pre-merge clear and loom's start refusal abort and redo the row's own unsettled merge-in and name the abort for another's.
