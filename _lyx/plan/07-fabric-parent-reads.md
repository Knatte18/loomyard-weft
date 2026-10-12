# Card 7 — fabric-parent-reads

**Create:**
- `internal/fabricengine#ParentRelation`
- `internal/fabricengine#ParentEqual`
- `internal/fabricengine#ParentBehind`
- `internal/fabricengine#ParentAhead`
- `internal/fabricengine#ParentDiverged`
- `internal/fabricengine#ParentNoUpstream`
- `internal/fabricengine#ParentTips`
- `internal/fabricengine#Fabric.ReadParentTips`
- `internal/fabricengine#Fabric.ChangedPaths`
- `internal/fabricengine#Fabric.HoldsWork`
- `internal/fabricengine#FastForwardRefusal`
- `internal/fabricengine#FastForwardDirty`
- `internal/fabricengine#FastForwardMergeInProgress`
- `internal/fabricengine#FastForwardNotFastForward`
- `internal/fabricengine#FastForwardResult`
- `internal/fabricengine#Fabric.FastForwardPair`
- `internal/fabricengine/parenttips_integration_test.go`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/gitrepo#Repo.Fetch`
- `internal/gitrepo#Repo.Upstream`
- `internal/gitrepo#Repo.IsAncestor`
- `internal/gitrepo#Repo.MergeTree`
- `internal/gitrepo#Repo.CommitTree`
- `internal/fabricengine#Fabric.mergeRecordExists`
- `internal/fabricengine#Fabric.warpWorktreeDirty`
- `internal/fabricengine#Fabric.warpUpstreamSHA`
- `internal/fabricengine#Fabric.ResetHard`
- `internal/fabricengine#Fabric.PullWeft`
- `internal/fabricengine#Fabric.weftHasUpstream`
- `internal/fabricengine#weftSHAOrEmpty`
- `internal/fabricengine#Fabric.recordWarpAdvance`
- `internal/fabricengine#NewMutations`
- `internal/fabricengine#MutationRecord`
- `internal/fabricengine#SyncOptions`

**Intent:**
Add the three code-side reads Merge-In and the moved-parent check need, in `fabricengine`, the one package that runs git on a pair (`PATTERN-fabric-git`, `PATTERN-gitexec-checked-call`).
Each reads the task pair's code repository, which shares its object store and remote-tracking refs with every worktree of the hub.

- `ReadParentTips(branch, fetch)` fetches the code repository's remotes first when `fetch` is true, then returns `ParentTips{Local, Upstream, Relation}`: `Local` is `refs/heads/<branch>`, `Upstream` the remote-tracking ref the branch is configured to track.
  With no upstream configured, or none fetched, `Upstream` is empty and `Relation` is `ParentNoUpstream`.
  Otherwise `Relation` is `ParentEqual`, `ParentBehind` (local is an ancestor of upstream, a fast-forward), `ParentAhead` (upstream is an ancestor of local) or `ParentDiverged`.
  A failed fetch is an error naming the reproducing command; a missing local branch is an error naming it.
- `ChangedPaths(from, to)` returns the sorted repo-relative paths `git diff --name-only from to` reports, the read `ReadConfigChanges` already makes with no pathspec.
- `HoldsWork(base, source)` reports whether merging `source` into `base` adds nothing: a clean `git merge-tree --write-tree` result whose tree equals `base`'s tree.
  A conflicted merge-tree holds nothing.
  It is the "parent already contains the task's work" test (D9) and holds after a squash landing, where ancestry does not.
- `FastForwardPair(opts)` moves the pair to its upstream only as a fast-forward, and never enters `Pull`'s rewrite path (hard reset or re-anchor on a non-fast-forward upstream).
  It is a mutating fabric verb under `PATTERN-mutation-record`: it accumulates a `*Mutations` record with `NewMutations` and returns it in `FastForwardResult`, which embeds `MutationRecord` and carries `Refusal FastForwardRefusal` and `NewHead string`.
  The record is returned even on a refusal, where it is empty.
  Its refusals, set in `Refusal`, carry neutral names a caller outside the fabric vocabulary's owner set may spell, each with a nil error and nothing moved:
  - a fabric merge record on the pair returns `FastForwardMergeInProgress`;
  - it fetches the code side once, and a dirty code worktree returns `FastForwardDirty`;
  - a local head that is not an ancestor of the freshly fetched upstream returns `FastForwardNotFastForward`.
  Once no refusal applies, it pulls the weft as `Pull`'s weft arm does, before the warp moves.
  A weft with no upstream (`weftHasUpstream` false) is skipped.
  A `PullWeft(opts)` failure is logged as a Warn and does not stop the warp side.
  A weft HEAD that moved, sampled before and after with `weftSHAOrEmpty`, appends `KindRepoAdvanced` at the weft path to the record.
  `opts.SkipGit` skips the weft side.
  Then it resets to the upstream through `ResetHard(rec, upstreamSHA)`, the arm `Pull`'s fast-forward branch uses, records the advance with `recordWarpAdvance` as `Pull` does, and sets `NewHead` to the upstream SHA.
  Any other failure is returned as the error.
  The ancestry check reads the upstream its own fetch returned, so an upstream force-pushed after the caller's read is refused, never rewritten onto the pair (D1).

`parenttips_integration_test.go` builds a hub with `hubforge` and asserts one table over `ReadParentTips`: equal, behind after a push to the upstream from another clone, ahead after a local commit, diverged after both, and no-upstream for a local-only branch, the push to the upstream seen only with `fetch` true.
It asserts `ChangedPaths` names exactly the files a commit changed, and `HoldsWork` true after the source's change was squash-committed onto the base and false before.
It asserts `FastForwardPair` moves a behind pair to its upstream tip, with `NewHead` the upstream SHA and the result's record holding the reset entry and, when the weft's upstream also moved, the weft advance entry.
It asserts that a fast-forward over a parent pair whose weft has no upstream moves the warp with a nil error.
It asserts `FastForwardPair` returns each refusal with a nil error, the pair unmoved and an empty record:
- `FastForwardDirty` when a tracked file is modified;
- `FastForwardMergeInProgress` when a fabric merge record is left on the parent pair by a conflicted fabric merge left unresolved;
- `FastForwardNotFastForward` when the upstream is force-pushed after `ReadParentTips`.

**ImpactSummary:** three new read-only `Fabric` methods and one fast-forward-only pair move; no existing fabric verb changes.

**Commit:** 7: fabricengine: parent tips, changed paths and the holds-work test
