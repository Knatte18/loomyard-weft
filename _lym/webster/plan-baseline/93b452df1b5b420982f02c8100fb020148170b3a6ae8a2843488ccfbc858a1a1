# Card 9 — merge-keeps-receiving-records

**Create:**
- `plan:internal/gitrepo#Repo.MergeStartNoFastForward` -> `func (r *Repo) MergeStartNoFastForward(ref string, squash bool) (MergeOutcome, error)`
- `plan:internal/fabricengine#Fabric.settleSourceRecords` -> `func (f *Fabric) settleSourceRecords(rec *Mutations, st *mergeState) ([]string, error)`
- `plan:internal/fabricengine#ErrMarkerChange` -> `type ErrMarkerChange struct`
- `plan:internal/fabricengine#refuseSourceMarkerChange` -> `func refuseSourceMarkerChange(repoDir, receiving, source string) error`
- `plan:internal/fabricengine#Fabric.SetAfterMergeStartForTest` -> `func (f *Fabric) SetAfterMergeStartForTest(hook func() error)`
- `plan:internal/fabricengine_test#TestMerge_OneRepository` -> `func TestMerge_OneRepository(t *testing.T)`

**Edit:**
- `internal/fabricengine#Fabric.Merge`
- `internal/fabricengine#Fabric.mergeIn`
- `internal/fabricengine#mergeState`
- `internal/fabricengine#pairDirtyReason`
- `internal/fabricengine#concludeMergeSides`
- `internal/fabricengine#MergeStateForTest`
- `internal/fabricengine_test#TestMergeState_Record`

**Extends:**
- `internal/fabricengine/doc.go`
- `internal/gitrepo/doc.go`

**Uses:**
- `internal/gitrepo#Repo.MergeStart`
- `internal/fabricengine#RecordsPathsAt`
- `internal/fabricengine#MergeTrailerKey`
- `internal/fabricengine#IsFabricMergeCommit`
- `internal/fabricengine#AnchorChangeTrailerKey`
- `internal/fabricengine#oneRepositoryAt`
- `internal/fabricengine#mergeInProgressReason`
- `internal/fabricengine#TestMergeVocabulary_ErrorsAreSideFree`
- `internal/hubforge#NewOneRepoHub`
- `internal/hubforge#AddPairWith`

**Intent:** D5: every fabric merge in a one-repository hub keeps the receiving side's records.
It covers `Fabric.Merge` with and without squash, which Finalize's local landing and a beam into its parent run, and `Fabric.mergeIn`, the merge-in from the parent.

Order in a one-repository worktree, the two-repository path unchanged:
0. The already-in-progress guard (`mergeInProgressReason`, the foreign-state check) runs first, before step 1 and before any dirtiness guard commits records, so a merge verb over a parked merge refuses as today and never aborts it.
   Way forward, unchanged: `lym fabric merge --abort`, or for a merge past step 3, `lym fabric merge --continue` once its conflicts are resolved.
1. Under the write lock, `commitRecordsPathsHeld` commits the receiving worktree's uncommitted records (D4).
   `pairDirtyReason` then judges code paths only, leaving out the paths `RecordsPathsAt(HEAD)` names.
   `refuseSourceMarkerChange` judges the source's own marker commits: those reachable from the source and not from the receiving HEAD that change `.lym-anchor` against their first parent,
   leaving out fabric merge commits (`IsFabricMergeCommit`: two parents and `MergeTrailerKey`), whose marker step 3 settled.
   It passes when `.lym-anchor` at the source tip equals the merge base's, or when no such commit exists,
   and otherwise refuses with `*ErrMarkerChange` when the newest such commit does not carry `AnchorChangeTrailerKey`.
   So a merge-in from the parent into a task at another anchor, after the task was merged back once, passes: the parent's merge commit kept its own marker and no source commit edited it.
   Its message names the marker change and stays free of side tokens (`TestMergeVocabulary_ErrorsAreSideFree`);
   way forward: commit on the source branch a `.lym-anchor` back at the merge base's value, or make the anchor change through `lym fabric anchor`, which commits it with the trailer, then merge again.
   Either step leaves no net change, or a newest marker commit carrying the trailer, so the next merge passes.
2. `MergeStartNoFastForward` merges without committing, adding `--no-ff` to the non-squash form, so a source that is a fast-forward of the receiver still makes a merge commit.
   It shares `MergeStart`'s body and outcome classification.
3. `settleSourceRecords` reads the records paths of both sides (`RecordsPathsAt` of HEAD and of the source),
   compares each path's index and worktree content against what step 2 wrote, and refuses as today's dirty refusal (`MergeGuardError` with the worktree-dirty reason and the paths) when one changed in between.
   That refusal leaves the merge parked with its records unsettled; its message names `lym fabric merge --abort`, then merge again, as the way forward (card 10's abort of an unsettled merge).
   Otherwise it sets those paths in index and worktree to HEAD's (`git restore --source=HEAD --staged --worktree`), deletes the ones only the source holds, and clears `mergeState.RecordsUnsettled` (new field, json `records_unsettled`).
   In a one-repository worktree the merge record is saved with `RecordsUnsettled` true before step 2, so a merge stopped between steps 2 and 3 reads unsettled;
   a two-repository record never sets it, and an older record reads it false, so both read settled.
   `MergeStateForTest` gains the field, `toMergeState` and `fromMergeState` carry it, and `TestMergeState_Record`'s `want` sets it true, so its no-zero-fields round-trip covers it.
4. The write lock is released before conflicts go to today's handling, and taken again for the conclude, where `concludeMergeSides` adds `MergeTrailerKey` to the commit message.

`Fabric.afterMergeStart` (new unexported field, `func() error`) runs between steps 2 and 3 when set; production leaves it nil.
A non-nil error it returns makes the merge verb return that error as it stands, with `MERGE_HEAD` and the unsettled merge record left in place, as a crash between the steps leaves them.
`SetAfterMergeStartForTest` sets it and lives in `export_test.go`.

`TestMerge_OneRepository` (tier1, `NewOneRepoHub`, parent and task pairs through `AddPairWith`) rows:
- `LocalMergeKeepsTheParentsRecords` and `SquashMergeKeepsTheParentsRecords`: uncommitted parent records plus conflicting records edits on both sides; after the merge the parent's records equal its own pre-merge records in index and worktree, and no task-only records file exists;
- `MergeInKeepsTheTasksRecords`;
- `FastForwardSourceMakesAMergeCommit`: the commit carries `MergeTrailerKey` and has two parents;
- `AnchorChangeMergedIntoTheParentChangesNoCodePath`;
- `TaskAddedAtAnotherAnchorMergesBack`: a task made with `--anchor` (card 6) merges into its parent without the marker refusal;
- `MergeInAfterABackMergeOfATaskAtAnotherAnchor`: the task of the row above, merged into its parent, is then merged-in from the parent and passes;
- `SourceMarkerEditRefuses`: the error is `*ErrMarkerChange` naming the marker change;
- `RevertedMarkerEditMerges`: after the row above, a commit restoring the marker on the source lets the merge pass;
- `RecordsWriteBetweenStepTwoAndThreeRefuses`: the hook writes a records path the source changed and returns nil; the merge refuses with the dirty refusal naming the path and `lym fabric merge --abort`, and the merge record stands with `RecordsUnsettled` true;
- `MergeOverAParkedMergeRefusesAndKeepsItsResolution`: a merge-in parked past step 3 with a resolved conflict; a second merge verb refuses as already in progress and the resolution stays in index and worktree.

**ImpactSummary:** Fabric.Merge and mergeIn gain one-repository steps, every fabric merge commit gains the trailer, the merge record gains a field an older record reads as unset, and gitrepo gains one method; two-repository merges are otherwise unchanged.
