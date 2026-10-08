# Card 12 — In-process "does HEAD hold this commit" read

**Create:**
- `plan:internal/gitrepo#Repo.HeadContains` -> `func (r *Repo) HeadContains(sha string) (bool, error)`
- `plan:internal/fabricengine#HeadContains` -> `func HeadContains(worktreePath, sha string) (bool, error)`

**Edit:**
- `internal/gitrepo/blobread_integration_test.go`

**Uses:**
- `internal/gitrepo#Repo.IsAncestor`
- `internal/fabricengine#Fabric.IsAncestor`

**Intent:**
Decisions 12 and 13 need to know whether the binary's VCS revision is an ancestor of a worktree's HEAD, computed in `internal/fabricengine` (`PATTERN-fabric-git`) as a local read in-process: no new OS process (`PATTERN-spawn-observability`), and go-git, not `gitexec`, for a local read (`PATTERN-gitrepo-client-boundary`).
`Repo.IsAncestor` shells out to `git merge-base` and so does not qualify.

`Repo.HeadContains` joins `blobread.go`'s go-git-only reads:
- An invalid hex `sha` returns `ErrInvalidSHA`.
- An unborn HEAD returns false.
- A `sha` whose commit object is absent from the local object store returns false with no error, looked up through `lookupObjectRetrying` so a freshly fetched pack is reindexed first.
- Otherwise it reports whether that commit is HEAD's commit or one of its ancestors.
- Its doc comment says it never runs `git`.
The absent-object answer settles Decision 12's open point: a revision the worktree has never fetched cannot be in its HEAD's history, so the worktree is behind the build, and no deploy advice follows.
`blobread.go`'s file header gains `HeadContains` in its list of reads.

`fabricengine.HeadContains(worktreePath, sha)` (in `warpforward.go`, beside `Fabric.IsAncestor`) is a thin delegation to `gitrepo.New(worktreePath).HeadContains(sha)`.
It takes a path rather than a `*Fabric`, because the stencil-seed pass runs from any worktree of the hub, a paired one or not.

Tests: `blobread_integration_test.go` gains a table test over a scratch repo: HEAD's own commit and an earlier commit are true; a commit only on an unmerged side branch is false; a well-formed sha present in no object store is false with no error; a malformed sha is `ErrInvalidSHA`; an unborn repository is false.
`fabricengine.HeadContains` gets no test of its own: card 14's `StencilSource` table, its only caller, drives it with the prime's HEAD sha and an absent sha.

**ImpactSummary:** Additive: one go-git read on gitrepo and one delegating function in fabricengine, with no caller yet.

**Commit:** 12: gitrepo, fabric: an in-process read of whether HEAD holds a commit
