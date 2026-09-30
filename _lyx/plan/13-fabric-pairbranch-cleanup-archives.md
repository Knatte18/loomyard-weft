# Card 13 — fabric-pairbranch-cleanup-archives

**Edit:**
- `internal/fabricengine#Topology.RemovePairBranch`
- `internal/fabricengine#PairBranchResult`
- `internal/fabricengine#Topology.Cleanup`
- `internal/fabricengine#CleanupBranchEntry`

**Create:**
- `internal/fabricengine/pairbranch_archive_integration_test.go`

**Uses:**
- `plan:internal/fabricengine#archiveWeftTip`
- `internal/fabricengine#deleteWeftBranch`
- `internal/fabricengine#deleteWeftBranchOnRemote`
- `internal/fabricengine#Topology.Prune`

**Intent:**
The other two paths that delete an existing pair's weft branch archive its tip first.

- `RemovePairBranch` calls `archiveWeftTip` after its refusals and before its first branch deletion, local or remote, for the pair's weft branch; the helper archives from origin's copy when only that copy remains.
  An archive error is returned, like its failed remote deletion, so batten's already-gone teardown arm halts resumable.
  `PairBranchResult` gains `ArchiveTag` and `ArchiveSkippedReason` with the same JSON keys as `RemoveResult`.
- `Cleanup`'s apply loop calls `archiveWeftTip` for each orphan weft branch before `deleteWeftBranch`, with the slug recovered from the branch (`WeftWarpSlug`, then the configured branch prefix stripped).
  An archive error fills the entry's `Error` and leaves that branch, local and remote, in place without aborting the sweep, matching how the loop treats other per-branch failures; `CleanupBranchEntry` gains `ArchiveTag`.
- `Prune` removes weft worktrees but deletes no weft branch (it leaves orphaned branches for `Cleanup`), so it needs no archive step; confirm this while implementing and state it in `Prune`'s doc comment beside the ownership rules.

Update the doc comments of `RemovePairBranch`, `Cleanup` and `CleanupBranchEntry` for the archive step.

Tests in `pairbranch_archive_integration_test.go`, over a hubforge hub with a weft origin: `RemovePairBranch` pushes the archive tag before deleting the branch locally and on origin, including when only the origin copy is left; an unreachable origin makes it return an error with the branch still present; `Cleanup` with apply archives each orphan weft branch it deletes, and an archive failure leaves that branch in place with the entry's `Error` set.

**ImpactSummary:** RemovePairBranch and Cleanup gain a network step before each weft-branch deletion and one result field each; Prune is unchanged.
