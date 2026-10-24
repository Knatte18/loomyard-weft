# Card 8 — weft-reset-executor

**Create:**
- `plan:internal/fabricengine#ResetRecordsToParent` -> `func ResetRecordsToParent(rec *Mutations, l *lyxcwd.Location, slug string) (ResetRecordsResult, error)`
- `plan:internal/fabricengine#ResetRecordsResult` -> `type ResetRecordsResult struct`
- `plan:internal/fabricengine_test#TestResetRecordsToParent` -> `func TestResetRecordsToParent(t *testing.T)`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#commitPendingRecords`
- `internal/fabricengine#ReadOriginFor`
- `internal/fabricengine#PathspecNames`
- `internal/fabricengine#ScopedPathspec`
- `internal/hubforge#CopyHub`
- `internal/hubforge#AddPairWith`

**Intent:**
D10 step 4's records reset, a destruction executor in `destroy.go` behind the shared check pipeline (PATTERN-fabric-git), declared in that file because it overwrites working-tree content.
`ResetRecordsToParent` resets the records content of slug's pair to what a fresh fork from its parent's records branch leaves, keeping the pair's origin record, in one commit on the pair's records branch, never rewriting history.
In order:
1. read the origin record (a missing one refuses naming `lyx fabric reconcile`); its `ParentBranch` names the parent records branch (`RecordsBranchName`), and an empty one refuses naming the pair as a root pair with no parent to reset to;
2. commit the records worktree's uncommitted record content under the record pathspec (`commitPendingRecords`, as `Remove` does before its archive tag), so the records history keeps it;
3. run the pipeline over a `pathRequest` on the records worktree: containment under the hub, ownership as the pair's own records worktree, and dirtiness as tracked or untracked changes outside the record pathspec, with no force;
   uncommitted content outside the pathspec refuses here naming the paths and, as the way forward, committing or discarding them in the records worktree and then re-running `lyx beam promote <slug>`;
4. replace the tracked content with the parent tip's tree minus the run-records root (`git read-tree` of the parent tip, `git checkout-index` over the worktree, removal of tracked files the parent lacks), restore the origin record as it was, and commit with the subject `fabric: reset records of <slug> to its parent`.
A pair whose content already equals that tree commits nothing and reports `Committed: false`, so a re-run after a stop finishes the reset.
`ResetRecordsResult` reports the pre-commit's SHA when one was made, the reset commit's SHA and `Committed`; the caller pushes through the pair's detached push.
`TestResetRecordsToParent` over a hubforge hub: a pair forked from the prime gains a committed records file and an uncommitted one under `_lyx`; after the reset the records worktree holds only the parent's content plus the origin record, both files are reachable in the records history (the uncommitted one in the pre-commit), and a second call commits nothing;
an untracked file outside the record pathspec refuses at step 3 naming its path and `lyx beam promote`, with no commit made;
a pair whose origin record the test deletes is refused naming `lyx fabric reconcile`, and a pair whose record the test rewrites with an empty `ParentBranch` is refused as a root pair, each with no commit made.

**ImpactSummary:** One new destruction executor in destroy.go; no existing executor or caller changes.
