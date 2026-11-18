---
round: 4
focus:
  - "D4 and D5 crash recovery: a merge fabric started that stopped before D5 step 3 leaves the source's records in the index and worktree, and `MergeAbort` runs `git reset --hard` through `resetMergeSides` (`internal/gitrepo/reset.go`). Settle whether the live records edits the transition wrote before the commit, such as the status file, survive the abort and reach the records commit, and whether the D4 acceptance row names a mechanism that can reach that outcome."
  - "D5's rule for a merge verb run again after a crash: `Fabric.Merge` and `Fabric.MergeIn` refuse with `mergeReasonAlreadyInProgress` in `internal/fabricengine/merge.go` when fabric's merge record exists, and only the resolver's `Resolve` aborts first. Settle whether the record keeps that refusal for a parked merge past step 3, whether it states the one abort-first path, and whether D4's pass-through of records commits over a parked merge agrees with D5."
  - "Any new text in D4 or D5 that introduces an abort, a HEAD or WarpStart move, or a skipped guard: settle whether the Decision states what it can skip or discard and what bounds it."
---

The two focus areas are where round 3's fixes landed; the rest of the record was settled.
