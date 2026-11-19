# Card 6 — one-repository-add

**Create:**
- `plan:internal/fabricengine_test#TestAdd_OneRepository` -> `func TestAdd_OneRepository(t *testing.T)`

**Edit:**
- `internal/fabricengine#Topology.Add`
- `internal/fabricengine#Topology.rollbackAdd`
- `internal/fabricengine#Healthy`
- `internal/fabricengine#Topology.repairPairWiring`
- `internal/fabricengine#checkJunctionHealth`
- `internal/fabricengine#Unwire`

**Extends:**
- `internal/fabricengine/doc.go`

**Uses:**
- `internal/fabricengine#createWeftWorktreeDroppingRuns`
- `internal/fabricengine#CommitRecordsPaths`
- `internal/fabricengine#WriteOrigin`
- `internal/fabricengine#oneRepositoryAt`
- `internal/fabricengine#AnchorChangeTrailerKey`
- `internal/hubforge#NewOneRepoHub`
- `internal/hubforge#AddPairWith`

**Intent:** D3's Add and the paths that no longer run in a one-repository hub.

`Topology.Add` in a one-repository hub (`oneRepositoryAt` of the start point's commit):
- creates no sibling worktree and no second branch: it skips `createGitWorktree` and creates the task worktree on the task branch through `createWeftWorktreeDroppingRuns`,
  which with card 3's geometry adds the worktree in the one repository at the task path, forked from the parent's tip,
  never writes the parent's run-records root (`shedrun.RunsRootRel` under the anchor) to disk, and reports whether the root is tracked;
- wires it through `WireJunctionsWith` (card 5's one-repository wiring);
- writes the origin record (`WriteOrigin`) and makes the first records commit on the task branch through `CommitRecordsPaths`,
  over the origin record, `.lym-anchor` and, when tracked, the run-records root, so the commit records the root's deletion and the marker;
  when `--anchor` wrote a new marker (`moveForkedWeftToAnchor`), that commit carries `AnchorChangeTrailerKey` naming the new anchor, so card 9's merge admits the change;
- skips the weft fast-forward, the records-worktree reuse and the remote weft-branch replacement, which name a second branch.
The adopt paths take the same route on the adopted branch.
`Topology.rollbackAdd` in that layout removes the one worktree and deletes the task branch only when Add created it.

`Healthy` raises none of the junction causes (`CauseJunctionMissing`, `CauseNotAJunction`, `CauseJunctionPointsElsewhere`) for a one-repository worktree, whose `_lym` and `.lym` are real directories;
`checkJunctionHealth` answers healthy for it, so `Topology.repairPairWiring` repoints nothing; `Unwire` reports nothing to unwire and changes nothing there.

`TestAdd_OneRepository` (tier1, `NewOneRepoHub` at anchor `backend`, pairs through `AddPairWith`) rows:
- `LeavesOneWorktreeWithRealRecordsDirectories`: `_lym` and `.lym` are real directories at the anchor, `.lym-anchor` is tracked at the worktree root and committed by Add's records commit, `.lym` is excluded, the exclude holds no `_lym` or weft artifact line, and no `weftname.Suffix` sibling or second branch exists;
- `ScratchLinksToTheWorktreesOwnDotLym`: `.scratch` at the anchor is a link to the worktree's own `.lym/scratch`;
- `ATaskForkedFromAParentCarryingRunRecordsStartsWithNone`: a parent whose branch commits a run directory under the run-records root forks a task whose disk and HEAD hold none;
- `AddedAtAnotherAnchorCarriesTheTrailer`: Add with `--anchor` at another anchor makes a first records commit carrying `AnchorChangeTrailerKey`;
- `AFailedWiringRollsBackTheOneWorktreeAndItsBranch`: Add failing at the wiring step, driven through the `beforeWire` seam as the `TestAddRollback_*` tests drive it, leaves no task worktree and no task branch;
- `AdoptingALocalBranchTakesTheSameRoute`: Add adopting a local branch made from the prime leaves one worktree on that branch with real records directories and no sibling;
- `ReconcileReportsNoJunctionCause`: `Healthy` of the pair is ok and `lym fabric reconcile` (`Topology.Reconcile`) records no junction repoint;
- `StatusAndConfigChangesReadTheOneBranch`: `Topology.Status` reports the pair with no branch mismatch, `Topology.RecordsBranchLive` reads the task branch live and `ReadConfigChanges` reads the task branch's config;
- `ResolveReadsTheAnchorFromTheMarker`: `lymcwd.Resolve` at the pair's anchored directory returns `AnchorRel` `backend`;
- `UnwireChangesNothing`.

**ImpactSummary:** Add, rollback, Healthy, reconcile repair and Unwire gain one-repository branches; their two-repository paths and callers are unchanged.
