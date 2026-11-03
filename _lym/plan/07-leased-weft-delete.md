# Card 7 — leased-weft-delete

**Edit:**
- `internal/fabricengine#removeWeftWorktree` -> `func removeWeftWorktree(rec *Mutations, l *lymcwd.Location, slug, branch string, force, alsoDeleteBranch, remote bool, branchPrefix, leaseSHA string) (weftTeardownResult, error)`
- `internal/fabricengine#Topology.removeSibling` -> `func (t *Topology) removeSibling(rec *Mutations, l *lymcwd.Location, slug, weftBranch string, pair pairState, force, remote bool, leaseSHA string) (weftTeardownResult, []string, error)`
- `internal/fabricengine#deleteWeftBranchOnRemote` -> `func deleteWeftBranchOnRemote(rec *Mutations, l *lymcwd.Location, branch, branchPrefix, weftRepoRoot, leaseSHA string, entry *CleanupBranchEntry)`
- `internal/fabricengine#Topology.removePair`
- `internal/fabricengine#Topology.rollbackAdd`
- `internal/fabricengine#Topology.Cleanup`
- `internal/fabricengine#weftTeardownResult`
- `internal/fabricengine#RemoveResult`
- `internal/fabricengine#CleanupBranchEntry`
- `internal/fabriccli#removeFields`
- `internal/fabriccli#TestRemoveFields`
- `internal/fabricengine_test#TestCleanup_ApplyArchivesEachOrphanBeforeDeleting`

**Create:**
- `internal/fabricengine#removeBeforeWeftDeleteHook`
- `internal/fabricengine#SetRemoveBeforeWeftDeleteHookForTest`
- `internal/fabricengine_test#TestRemove_LeasedWeftDelete`

**Extends:**
- `internal/fabricengine/doc.go`
- `internal/fabriccli/doc.go`

**Uses:**
- `internal/gitrepo#ErrLeaseRejected`

**Intent:** Decision record D6: every deletion of a slug's weft branch on origin is leased on the origin tip the deleter archived.

- `removeWeftWorktree` and `deleteWeftBranchOnRemote` take `leaseSHA` and pass it to `remoteBranchRequest.leaseSHA`, so the delete runs `--force-with-lease=refs/heads/<branch>:<lease>`.
  The empty-lease check runs after `removeWeftWorktree`'s existing no-origin check, so a records repository with no origin still reports its `remoteSkippedReason`.
  With an origin, an empty `leaseSHA` means origin held no weft branch when the archive read it, and the remote delete is skipped with nothing reported, since there is nothing on origin to delete; `rollbackAdd` passes `""` with `remote` false, as it deletes nothing on origin today.
- In a removal the lease is card 6's `weftArchive.leaseTip`: the local tip when the archive step's branch push landed, else the origin tip it read; an archive tag covers that tip.
  `Topology.removePair` passes it to `removeSibling`, which passes it to `removeWeftWorktree`, and in its origin-only path to `deleteWeftBranchOnRemote`, never reading origin a second time.
- `Topology.Cleanup` passes the archive's `leaseTip`, the origin tip the sweep read and archived.
- `removeBeforeWeftDeleteHook`, nil in production, runs right before each leased weft delete so a test can move origin's branch between the read and the delete.
- A delete the lease refuses, `errors.Is(err, gitrepo.ErrLeaseRejected)` through card 1, keeps the branch on origin and is reported as kept, never as a failure: `weftTeardownResult` gains `remoteBranchKeptReason`, `RemoveResult` gains `RemoteBranchKeptReason` (`remote_branch_kept_reason`, omitempty), `CleanupBranchEntry` gains `RemoteKeptReason` (`remote_kept_reason`, omitempty), each reading `kept: origin's weft branch moved during the removal, so it is another process's live branch` with the branch named; `RemoteBranchError` and `RemoteError` stay for every other failure.
  `removeFields` adds `remote_branch_kept_reason` when set; `TestRemoveFields` gains that row.
- Bound: a branch kept this way was pushed by another process between the read and the delete, which under D7 is no new pair of this slug on this machine; a later `Add` adopting it is correct.
- Extend `internal/fabricengine/doc.go`'s remove and cleanup paragraphs with the lease and the kept report, and `internal/fabriccli/doc.go`'s `--remote` sentence with `remote_branch_kept_reason`.

Test `TestRemove_LeasedWeftDelete`, `tier1`, one hub per row, removing with `remote` true:
- `SetRemoveBeforeWeftDeleteHookForTest` pushes a new commit to origin's weft branch from another clone: the removal succeeds, origin still holds the branch at the moved tip, `RemoteBranchDeleted` is false and `RemoteBranchKeptReason` is set.
- Origin holds no weft branch and an `update` hook refuses the archive step's branch push: the removal succeeds, no remote delete is attempted, and `RemoteBranchError` and `RemoteBranchKeptReason` are empty.
- Origin lacks the local weft tip at the removal: the archive step pushes it and the delete removes origin's branch, the lease on the pushed tip (the hook records `ls-remote` showing origin at the local tip right before the delete).
- An origin-only weft branch (no local branch or worktree): deleted under the lease on the tip the archive read; moved by the hook, kept and reported.
The delete of an unmoved branch at the read tip stays covered by `TestRemove_RemoteTrueDeletesWeftBranchOnRemote`, which runs the leased path once this card lands, unchanged.
`TestCleanup_ApplyArchivesEachOrphanBeforeDeleting`'s diverged-orphan row from card 6 runs with `remote` true and asserts origin's weft branch deleted; a further row moves origin's branch through the hook and asserts `RemoteKeptReason` set and the branch kept.

**ImpactSummary:** Re-signs three private helpers whose every caller (`removePair`, `removeSibling`, `rollbackAdd`, `Cleanup`) is on this card; adds one omitempty kept-reason field to each of three result types and one envelope field, with no change to the error return of `Remove` or `Cleanup`.

**Commit:** `7: Every weft delete on origin is leased on the tip its archive read`
