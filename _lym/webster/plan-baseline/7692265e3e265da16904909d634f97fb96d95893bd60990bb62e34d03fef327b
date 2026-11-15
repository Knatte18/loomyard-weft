# Card 4 — fabric-origin-reads

**Create:**
- `plan:internal/fabricengine#OriginParentHoldsTaskWork` -> `func OriginParentHoldsTaskWork(l, pairLocation *lymcwd.Location, slug string) (bool, error)`
- `plan:internal/fabricengine#OriginRunStatus` -> `func OriginRunStatus(prime *lymcwd.Location, slug string) (status shedengine.Status, found bool, err error)`
- `plan:internal/fabricengine#FetchedTaskBranchOnOrigin` -> `func FetchedTaskBranchOnOrigin(prime *lymcwd.Location, slug string) (bool, error)`
- `plan:internal/fabricengine#TestOriginRunReads` -> `func TestOriginRunReads(t *testing.T)`

**Uses:**
- `internal/fabricengine#ParentHoldsTaskWork`
- `internal/fabricengine#UpstreamRunUnfinished`
- `internal/fabricengine#Topology.TaskBranchOnOrigin`
- `internal/fabricengine#TaskBranchSlugs`
- `internal/fabricengine/doc.go`

**Intent:** The three reads from origin's branches that D1 needs, inside fabric as `PATTERN-fabric-boundary` and `PATTERN-gitrepo-client-boundary` require.
None fetches: each reads the remote-tracking refs the last fetch left, so it makes no network call.

- `OriginParentHoldsTaskWork` is `ParentHoldsTaskWork` against `refs/remotes/origin/<parent branch>` in place of `refs/heads/<parent branch>`:
  a landing run is past its parent-side merge only when origin's parent branch, as this machine last fetched it, holds the task's work.
  It shares `parentHoldsTaskWork`'s body through a ref argument, so the two differ only in the ref.
  A parent branch with no remote-tracking ref reads false.
- `OriginRunStatus` reads the child run record of slug, `_lym/shed/self/status.json` under the pair's anchor as `shedrun.TreeRunFileRels` names it,
  from the tree of the remote-tracking ref of slug's records branch, for a machine that holds no pair for the run.
  It finds the anchor and the slug-or-`self` directory as `UpstreamRunUnfinished` does, reads through the prime's records repository, and decodes with `shedengine.ParseStatusTolerant`.
  found is false when the ref or the file is absent.
- `FetchedTaskBranchOnOrigin` reports whether slug's task branch has a remote-tracking ref under origin in the prime's warp repository.
  It matches the branch under the current branch prefix, or the bare slug of a pair made before the prefix was set, as `Topology.TaskBranchOnOrigin` does,
  but reads the refs in-process as `TaskBranchSlugs` does, where `TaskBranchOnOrigin` runs `git ls-remote`.
  Bound: a task branch deleted on origin since the last fetch still reads present until the next fetch drops its ref.

`TestOriginRunReads` is tier1, over a `hubforge` hub with a pushed pair:
rows for a parent branch whose local tip holds the task's work while origin's fetched tip does not (`OriginParentHoldsTaskWork` false, `ParentHoldsTaskWork` true),
for both holding it (true), for a child status pushed and read with the pair removed locally (found with its state), and for a slug with no records branch on origin (found false);
for `FetchedTaskBranchOnOrigin`, a pushed task branch reads true and a slug never pushed reads false,
and a row that points origin's URL at a missing path after the fetch still reads the pushed branch true, which shows the read makes no network call.
