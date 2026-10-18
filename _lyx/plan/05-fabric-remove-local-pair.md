# Card 5 — fabric-remove-local-pair

**Create:**
- `plan:internal/fabricengine#RemoveStepRecordsDropped` -> `const RemoveStepRecordsDropped = "records_dropped"`
- `plan:internal/fabricengine#dirtyRecordsDisposable` -> `func dirtyRecordsDisposable() pathDirtiness`
- `plan:internal/fabricengine#removeLocalWeft` -> `func removeLocalWeft(rec *Mutations, l *lyxcwd.Location, slug, branch, branchPrefix string) (steps []string, err error)`
- `plan:internal/fabricengine_test#TestRemove_LocalPair` -> `func TestRemove_LocalPair(t *testing.T)`
- `plan:internal/fabricengine_test#TestOriginWrites_LocalMarkedBranch` -> `func TestOriginWrites_LocalMarkedBranch(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology.Remove`
- `internal/fabricengine#RemoveResult`
- `internal/fabricengine#pairState`
- `internal/fabricengine#Topology.inspectPair`
- `internal/fabricengine#archiveWeftTip`
- `internal/fabricengine#pathDirtinessKind`
- `internal/fabricengine#checkPathDirtiness`
- `internal/fabricengine#checkRemoteBranchRequest`
- `internal/fabriccli#removeFields`
- `internal/fabriccli#TestRemoveFields`
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/gitrepo#Repo.BranchMarkedLocal`
- `internal/fabricengine#Topology.AddLocal`
- `internal/fabricengine#deleteWarpBranch`
- `internal/fabricengine#ArchiveWeftTipForTest`
- `internal/fabricengine#CheckRemoteBranchRequestForTest`

**Intent:** Make `lyx fabric remove` of a local pair run no origin step and drop its records (D3's remove, its bound and the hand-deleted-branch rule).

`pairState` gains `localMark`, the branch carrying the mark: `inspectPair` reads `BranchMarkedLocal` of the warp branch in the warp repo, then of the weft branch in the weft repo root, and keeps the first that is marked.
A pair counts as local when either surviving branch carries the mark, so a pair one of whose branches was deleted by hand is still local; a pair with no surviving mark is removed as an ordinary pair, unchanged.

`Remove` runs `RemoveRefusal` first, as today: every existing refusal (slug, prime, nothing-remains, merge-in-progress, dirtiness) still runs, and `force` still answers dirtiness only.
On a local pair it then:
- skips `commitPendingRecords` and `archiveWeftTip`: nothing is committed or archived, and no tag is pushed;
- reads the local weft branch's tip, when the branch survives, into `RemoveResult.DroppedRecordsTip`;
- tears down portal, launchers, junctions and the task worktree as today (a local pair has no portal or launcher, so those steps record nothing);
- removes the sibling through `removeLocalWeft`, which removes the weft worktree through `removeGitWorktree` with `dirtyRecordsDisposable()` as its dirtiness, so uncommitted records are discarded without `--force`, deletes the local weft branch through `deleteBranch`, prunes, and reports `RemoveStepSiblingWorktree`, `RemoveStepSiblingBranch` and `RemoveStepRecordsDropped` for what changed; it never calls `RemoteURL`, `remoteHeadTip` or `deleteRemoteBranch`;
- deletes the warp branch through `deleteWarpBranch` with today's gate, so a branch with unlanded work is kept with `CodeBranchKeptReason` and the remove still succeeds;
- never calls `deleteTaskBranchOnOrigin`, and with `remote` true sets `RemoteSkippedReason` to "the pair is local-only: nothing of it is on origin, so --remote has nothing to do".
`RemoveResult` gains `Local bool` (`json:"local,omitempty"`), `LocalMarkBranch string` (`json:"local_mark_branch,omitempty"`) and `DroppedRecordsTip string` (`json:"dropped_records_tip,omitempty"`), so a hand-set mark on an ordinary pair shows in the result.
So no new refusal guards a local remove and no flag answers one; what the path skips is the archive, the pending-records commit and every origin call, and what bounds it is the mark, which only `lyx fabric add --local` writes.

`dirtyRecordsDisposable`, in `destroy.go` (PATTERN-fabric-destruction-chokepoint), declares a new `pathDirtinessKind` whose dirtiness step passes: the records on that worktree are disposable by the local pair's contract, and the refusal for content outside the record pathspec has already run in `RemoveRefusal`.
`archiveWeftTip` returns no tag, the skipped reason "the branch is local-only, so it is not archived" and no error when the local weft branch carries the mark, before it resolves a tip or touches origin.
`checkRemoteBranchRequest` refuses with a `CheckOwnership` destructive refusal, "the branch is local-only, never on a remote; nothing to delete; remove the pair with `lyx fabric remove <slug>`, which runs no origin step", when the local copy of `req.branch` in `req.repoDir` carries the mark.
Neither of those two checks is reached by `Remove`'s local path, which calls neither.
`archiveWeftTip`'s check guards any caller that archives a marked branch; card 6's `Cleanup` reads the mark before it would call it.
The gate's check is a second line behind `Remove`'s local path here and `Cleanup`'s mark read in card 6: it reads the local copy of the branch, so it refuses a caller that would delete the remote copy of a branch whose marked local copy still stands, and `TestOriginWrites_LocalMarkedBranch` reaches it.
`removeFields` maps the three new fields to `local`, `local_mark_branch` and `dropped_records_tip`, each present only when set.

`TestRemove_LocalPair` runs over a hubforge hub whose `origin` URLs point at a path that does not exist, so any origin step fails the remove, with one `AddLocal` pair per row, each removed without `--force`:
- a fresh pair: removed, `Local` true, `LocalMarkBranch` the warp branch, `DroppedRecordsTip` the weft tip, no `archive/` tag in the weft repo, the weft branch gone;
- a pair whose weft side carries a committed run record (a shed status file committed with `CommitRecordsPaths`): removed and its records gone with the branch;
- a pair with an uncommitted record inside the pathspec: removed, nothing committed;
- the fresh pair removed with `remote` true: removed, `RemoteSkippedReason` naming local-only;
- a pair with an unlanded warp commit: removed, the warp branch kept with a non-empty `CodeBranchKeptReason` and still marked;
- a pair whose weft worktree and branch were deleted by hand: still removed as local, `LocalMarkBranch` the warp branch, with no origin step;
- a pair whose task worktree and warp branch were deleted by hand: still removed as local, `LocalMarkBranch` the weft branch, with no origin step.
An ordinary pair's remove stays pinned by the existing remove tests, unchanged.
`TestOriginWrites_LocalMarkedBranch`, an external `fabricengine_test` test, since an internal test cannot import hubforge without an import cycle, runs over a hubforge hub with an `AddLocal` pair and the weft `origin` URL pointing at a missing path, and has two rows: `archiveWeftTip`, driven through `ArchiveWeftTipForTest`, over the marked weft branch returns no tag, the local-only skipped reason and a nil error, and creates no `archive/` tag; and the gate, driven through `CheckRemoteBranchRequestForTest` over a request deleting that branch, returns the `CheckOwnership` refusal naming local-only and `lyx fabric remove`.
`TestRemoveFields` gains a row with the three fields set and a row asserting them absent when unset.

`internal/fabricengine/doc.go` describes the local remove path and the records-disposable rule.

**ImpactSummary:** `Remove` gains one branch taken only for a marked pair; the gate's two new checks fire only on marked branches, so every ordinary remove, cleanup and add replacement is unchanged.

**Commit:** 5: fabric remove: a local pair runs no origin step and drops its records
