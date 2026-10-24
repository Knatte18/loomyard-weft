# Card 8 — weft-reset-executor

**Create:**
- `internal/fabricengine#ResetRecordsToParent`
- `internal/fabricengine#ResetRecordsResult`
- `internal/fabricengine_test#TestResetRecordsToParent`

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
1. read the origin record; a missing one refuses as `MarkBeam`'s does (card 2), naming `lyx fabric reconcile` for a missing records worktree and, for a record file absent from a present records worktree, restoring it from the records history in that worktree, with `git checkout HEAD -- <record path>` while the deletion is uncommitted, else from the commit before the one that deleted it, and then re-running the verb; its `ParentBranch` names the parent records branch (`RecordsBranchName`), and an empty one refuses naming the pair as a root pair with no parent to reset to and, as the way forward, a project entry through `lyx board upsert` under a slug of its own and `lyx beam create` on it, since a root pair cannot become a beam;
2. commit the records worktree's uncommitted record content under the record pathspec (`commitPendingRecords`, as `Remove` does before its archive tag), so the records history keeps it;
3. run the pipeline over a `pathRequest` on the records worktree: containment under the hub, ownership as the pair's own records worktree, and dirtiness as tracked or untracked changes outside the record pathspec, with no force;
   uncommitted content outside the pathspec refuses here naming the paths and, as the way forward, committing or discarding them in the records worktree and then re-running `lyx beam promote <slug>`;
4. replace the tracked content with the parent tip's tree minus the run-records root (`git read-tree` of the parent tip, `git checkout-index` over the worktree, removal of tracked files the parent lacks), restore the origin record as it was, and commit with the subject `fabric: reset records of <slug> to its parent`.
A pair whose content already equals that tree commits nothing and reports `Committed: false`, so a re-run after a stop finishes the reset.
`ResetRecordsResult` reports the pre-commit's SHA when one was made, the reset commit's SHA and `Committed`; the caller pushes through the pair's detached push.
`TestResetRecordsToParent` over a hubforge hub: a pair forked from the prime gains a committed records file and an uncommitted one under `_lyx`; after the reset the records worktree holds only the parent's content plus the origin record, both files are reachable in the records history (the uncommitted one in the pre-commit), and a second call commits nothing;
an untracked file outside the record pathspec refuses at step 3 naming its path and `lyx beam promote`, with no commit made;
a pair whose origin record the test deletes from its present records worktree is refused naming `git checkout HEAD -- <record path>` in that worktree, a pair whose records worktree the test removes is refused naming `lyx fabric reconcile`, and a pair whose record the test rewrites with an empty `ParentBranch` is refused as a root pair, its text asserted to carry both `lyx board upsert` and `lyx beam create`, not only the words root pair, each with no commit made.

**ImpactSummary:** One new destruction executor in destroy.go; no existing executor or caller changes.
