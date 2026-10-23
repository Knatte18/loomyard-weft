# Card 5 — task-branch-discard

**Create:**
- `plan:internal/fabricengine#DiscardResult` -> `type DiscardResult struct`
- `plan:internal/fabricengine#Topology.DiscardTaskBranch` -> `func (t *Topology) DiscardTaskBranch(l *lyxcwd.Location, slug string) (DiscardResult, error)`
- `plan:internal/fabricengine#archiveTaskTip` -> `func archiveTaskTip(rec *Mutations, repoDir, slug, tip string) (string, error)`
- `plan:internal/fabricengine#deleteArchivedTaskBranch` -> `func deleteArchivedTaskBranch(rec *Mutations, repoDir, warpBranch, localTag, originTag, leaseTip string) (localDeleted, originDeleted bool, err error)`
- `plan:internal/fabricengine#branchDirtinessArchivedTip` -> `const branchDirtinessArchivedTip branchDirtinessKind = iota`
- `plan:internal/fabricengine#dirtyArchivedTip` -> `func dirtyArchivedTip(archiveTag string) branchDirtiness`
- `plan:internal/fabricengine_test#TestDiscardTaskBranch` -> `func TestDiscardTaskBranch(t *testing.T)`
- `plan:internal/fabricengine_test#TestDeleteArchivedTaskBranch` -> `func TestDeleteArchivedTaskBranch(t *testing.T)`
- `plan:internal/fabricengine#DeleteArchivedTaskBranchForTest` -> `func DeleteArchivedTaskBranchForTest(repoDir, warpBranch, localTag, originTag, leaseTip string) (localDeleted, originDeleted bool, err error)`

**Edit:**
- `internal/fabricengine#checkBranchDirtiness`
- `internal/fabricengine#checkRemoteBranchRequest`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#deleteBranch`
- `internal/fabricengine#deleteRemoteBranch`
- `internal/fabricengine#dirtyArchivedOnRemote`
- `internal/fabricengine#ArchiveTagPrefix`
- `internal/fabricengine_test#TestDeleteArchivedWeftBranch_StaleLeaseFailsAndKeepsTip`

**Intent:**
Give fabric the one removal `--remove` needs, D5's archive-and-delete of the task branch, in fabric's own vocabulary so callers outside the owner set never name a side.

`Topology.DiscardTaskBranch` archives and deletes the slug's task branch locally and on the warp origin whether or not its work is landed (D5).
It reads the local tip (`refs/heads/<prefix><slug>`) and the origin tip (`remoteHeadTip`, then a fetch of that branch so its objects are local); neither present returns a zero result and no error.
When one tip contains the other (`isAncestorOrEqual`) only the containing tip is archived; otherwise both are.
`archiveTaskTip` tags a tip `archive/<slug>/<first 12 hex>` in the warp repo, reusing a tag already at that tip and refusing one of that name elsewhere, and pushes it to the warp origin with `gitrepo.PushWithRetry` on `archiveTagPushDelays()`, as `archiveWeftTip` does; every error wraps `ErrArchiveFailed`, and `KindTagPushed` is recorded only after the push succeeded.
A failed tag push returns its error with no branch deleted, so the caller's re-run retries.
Then `deleteArchivedTaskBranch` deletes the origin branch through `deleteRemoteBranch` with `dirtyArchivedOnRemote(originTag)`, leased to the observed origin tip, and the local branch through `deleteBranch` with the new `dirtyArchivedTip(localTag)`, both with `ownedPairWarpBranch(warpBranch, "")` ownership; an empty tag skips that side, and destruction stays in `destroy.go`'s executors (PATTERN-fabric-destruction-chokepoint).
A lost lease or a refused gate is returned as an error naming the branch, never a kept reason, since the cancel's caller must not report a branch it did not delete.
`DiscardResult` embeds `MutationRecord` (PATTERN-mutation-record) and carries `ArchiveTags []string`, `LocalDeleted` and `OriginDeleted`.

`branchDirtinessArchivedTip` is appended as the last line of the existing `branchDirtinessKind` iota block, after `branchDirtinessUnlandedRemoteTip`, so it takes the next value of the block and never `branchDirtinessUnset`'s; the Create head's `= iota` stands for that position, not a declaration of its own.
`checkBranchDirtiness` accepts it for a local delete only when the tag is non-empty and the branch's current tip is an ancestor of or equal to the tag's target, and refuses otherwise with a `destructiveRefusal` naming the uncovered tip; `checkRemoteBranchRequest` refuses it as answering a local question.
What it lets through: a local delete of an unlanded task branch, bounded by an archive tag on the warp origin holding its tip, which only `DiscardTaskBranch` constructs.

`TestDiscardTaskBranch` is a tier-1 integration test over a hubforge hub: a task branch with an unlanded commit locally and a different unlanded commit on origin gets two archive tags on the warp origin and is gone in both places; a local tip contained in the origin tip gets one tag, and an origin tip contained in the local tip gets one tag; no branch anywhere returns a zero result; with the warp origin reachable but a `pre-receive` hook on its bare repository rejecting `refs/tags/archive/*`, the call returns an error wrapping `ErrArchiveFailed` with the branch still present locally and on origin.
`TestDeleteArchivedTaskBranch` is a tier-1 integration test in the shape of the `TestDeleteArchivedWeftBranch_*` tests, reaching `deleteArchivedTaskBranch` through `DeleteArchivedTaskBranchForTest`, a wrapper in `export_test.go` beside `DeleteArchivedWeftBranchForTest`: a local tag that does not hold the branch's tip refuses the local delete with the branch kept; `dirtyArchivedTip` on a remote delete is refused by `checkRemoteBranchRequest`; a lease tip the origin no longer sits at fails, keeps the origin tip and is returned as an error.
`doc.go` names the archive-and-delete beside Remove's paragraph.

**ImpactSummary:** Two destroy-gate checks gain one kind each, reached only by the new constructor; every existing branch deletion keeps its declared kind and behaviour.
