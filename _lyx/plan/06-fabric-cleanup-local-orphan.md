# Card 6 — fabric-cleanup-local-orphan

**Create:**
- `plan:internal/fabricengine_test#TestCleanup_LocalOrphanPushesNothing` -> `func TestCleanup_LocalOrphanPushesNothing(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology.Cleanup`
- `internal/fabricengine#CleanupBranchEntry`

**Uses:**
- `plan:internal/gitrepo#Repo.BranchMarkedLocal`
- `plan:internal/fabricengine#Topology.AddLocal`
- `internal/fabricengine#deleteWeftBranch`

**Intent:** Make `lyx fabric cleanup --apply` over a local pair's orphan weft branch push nothing (D3: "pushes no archive tag and deletes nothing remote; the branch itself goes by the records rule").

Under `apply`, `Cleanup` reads each orphan branch's mark with `BranchMarkedLocal` before it archives.
A marked orphan is not archived, so no tag is pushed, and is deleted locally through `deleteWeftBranch` as a disposable records branch; `deleteWeftBranchOnRemote` is not called for it, `remote` or not.
`CleanupBranchEntry` gains `Local bool` (`json:"local,omitempty"`), set on such an entry, so the envelope says why it carries no archive tag.
`archiveWeftTip`'s and the remote gate's own mark checks (card 5) stay as a second line behind this branch.
A dry run reports a marked orphan as today, with `Local` set.

`TestCleanup_LocalOrphanPushesNothing` makes a local pair with `AddLocal`, removes both its worktrees by hand (`git worktree remove` in each repo), so its weft branch is an orphan checked out nowhere, which `Cleanup` would otherwise mark `Protected`, and points the weft repo's `origin` URL at a path that does not exist.
A dry run `Cleanup(apply=false, remote=true)` reports the branch's entry with `Local` set and deletes nothing; then `Cleanup(apply=true, remote=true)` returns nil with the entry `Deleted` and `Local`, no `ArchiveTag`, no `RemoteError`, no `archive/` tag in the weft repo, and the local branch gone.

**ImpactSummary:** One extra local config read per orphan under apply; ordinary orphans keep today's archive-then-delete order.

**Commit:** 6: fabric cleanup: a local pair's orphan weft branch pushes nothing
