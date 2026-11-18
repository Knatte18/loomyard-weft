```yaml
lens: fix-way-forward
outcome: reported
```

Fork lens fix-way-forward: check that each refusal, way forward and row the fix introduced can be carried out from the state it names and leads to progress.

Ways forward the fix introduced:
1. A merge verb re-run over a merge that stopped before step 3 or a parked merge refuses as already in progress. Way forward: `lym fabric merge --abort`, or `lym fabric merge --continue` for a merge past step 3. Both flags exist (`internal/fabriccli/merge_verbs.go:290-291`), and a merge task's writer may run them (`internal/fabriccli/roleguard.go` `mergeTaskVerb`). Reachable, and both make progress.
2. `lym fabric merge --continue` on a merge stopped before step 3 refuses. Way forward: `--abort`, then merge again. `--abort` reaches D4's commit-then-abort rule, which leaves a clean worktree, so merging again is reachable.
3. The new D4 and D5 acceptance rows each name a state a test can build: a crash before step 3, a parked merge past step 3, and `--continue` before step 3.

Findings:

- MEDIUM. Changed text: D4, "they move HEAD and `WarpStart` onto it as above, and only then abort the merge. The abort resets hard to that commit". The record never says the abort runs under the write lock the records commit holds, so the step cannot be carried out safely as written.
  - The step after a records commit is an abort through `fabricengine.Fabric.MergeAbort`, which takes the weft write lock itself (`internal/fabricengine/mergelifecycle.go:348`).
  - D4 says the records commit holds that same lock.
  - If the commit keeps the lock while calling `MergeAbort`, the second acquire blocks unless the lock is re-entrant.
  - If it releases the lock first, a records write can land between the commit and the abort, and `resetMergeSides`' hard reset drops it (`internal/fabricengine/destroy.go:2492-2501`). That is the loss F30 set out to remove, and it falls outside the record's stated bound, which covers only paths the source changed.
  - Fix direction: state that the commit and the reset run under one hold of the write lock, through an abort path that does not take the lock again.

- LOW, outside this lens. Changed text: the D4 acceptance row, "in HEAD, the worktree and origin and no source records in either". "Either" covers three places; "in any of them" says what is meant.

Tool calls:
- Read `internal/fabriccli/roleguard.go` lines 1-60 and `internal/fabriccli/merge_verbs.go` lines 175-215, to confirm that `--continue` and `--abort` exist and that the roles which would follow the way forward may run them. My context showed only the grep line numbers for those flags.
