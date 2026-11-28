# Card 3 — fabric-landing-chain

**Create:**
- `plan:internal/fabricengine#ChainLink` -> `type ChainLink struct`
- `plan:internal/fabricengine#ChainCommit` -> `type ChainCommit struct`
- `plan:internal/fabricengine#ChainResult` -> `type ChainResult struct`
- `plan:internal/fabricengine#Fabric.BuildLandingChain` -> `func (f *Fabric) BuildLandingChain(base string, links []ChainLink) (ChainResult, error)`
- `plan:internal/fabricengine#Fabric.MergesClean` -> `func (f *Fabric) MergesClean(base, source string) (bool, error)`
- `plan:internal/fabricengine#Fabric.MoveCodeTo` -> `func (f *Fabric) MoveCodeTo(base, tip string) (bool, error)`
- `plan:internal/fabricengine#NewestLandingOf` -> `func NewestLandingOf(l *lymcwd.Location, branch, slug string) (string, bool, error)`
- `internal/fabricengine/landingchain_integration_test.go`

**Uses:**
- `internal/fabricengine/doc.go`
- `internal/fabricengine#Fabric.HoldsWork`
- `internal/fabricengine#landingCommitsOn`
- `internal/fabricengine#MergeGuardError`
- `internal/gitrepo#Repo.MergeTree`
- `plan:internal/gitrepo#Repo.WriteCommitTree`

**Intent:** Add fabric's batch-landing verbs (D4, and D6's guarded move), in a new file `internal/fabricengine/landingchain.go` beside `merge.go` and `mergelifecycle.go`, so `fabric-core-merge-landing` moves them with today's merge code.
They take plain values and know nothing of the landing queue's records (`PATTERN-fabric-boundary`'s isolation rule).

- `ChainLink` holds `Slug`, `TaskHead`, `Message` (the whole landing message, trailer included) and `TrueMerge`.
  `ChainCommit` holds `Slug` and `Commit`.
  `ChainResult` holds `Commits []ChainCommit` (the chain, in order), `Conflicted []string` and `Held []string` (slugs left out).
- `BuildLandingChain(base, links)` runs on the pair's code repository and writes no ref, worktree or index.
  It resolves `base` and each task head to a SHA first, as `HoldsWork` does with `ResolveSHA`; a ref that does not resolve returns the error naming the slug, or naming `base`.
  For each link in order it first asks `HoldsWork(prev, link.TaskHead)`, `prev` being the last chain commit or `base`.
  A link the chain already holds goes to `Held` and adds no commit, so no empty second landing commit is written.
  Otherwise it runs `MergeTree(prev, link.TaskHead)`; a merge that is not clean puts the slug in `Conflicted`, and the next link merges onto the same `prev`.
  A clean merge writes the landing commit with `WriteCommitTree(tree, parents, link.Message)`: parents `[prev]` when the link squashes, `[prev, link.TaskHead]` when it true-merges.
- `MergesClean(base, source)` resolves both refs to SHAs as `HoldsWork` does, then reports whether `MergeTree(base, source)` is clean, so a caller can tell a merge-in that needs a conflict session before running it (D8).
  A caller may pass branch names, such as the parent branch and the task branch.
- `MoveCodeTo(base, tip)` is the guarded move of D6 on the pair whose handle it is.
  It reads the checked-out branch's head; a head other than `base` returns false and moves nothing.
  A code checkout with tracked dirt, or a detached head, is refused with a `*MergeGuardError` whose `WorktreeDirty` or `DetachedHead` accessor answers, carrying the checkout and the dirty paths as the merge guard's does, so a caller words it as today's parent-side merge refusal.
  A tip that does not descend from `base` returns an error naming both, and moves nothing.
  Otherwise it fast-forwards the checked-out branch and its worktree to `tip` with `git merge --ff-only`, through `gitrepo`, and returns true.
  It never moves the records side.
- `NewestLandingOf(l, branch, slug)` returns the newest commit on `branch`'s first-parent history in the repository at `l` whose `Lym-Landing` trailer names `slug`, origin's commits included, and whether one was found.
  It reads trailers as `landingCommitsOn` does, but walks the whole first-parent chain rather than stopping at the tracking ref; it walks at most `maxLandingWalk` commits, a named constant of 500, and reports none past that.

`landingchain_integration_test.go` is tier1, in `package fabricengine_test`, over a `hubforge` hub with three task pairs off one parent pair.
`TestBuildLandingChain` rows: three branches squashed build three commits, each with one parent, the previous one first, its run's message and `Lym-Landing` trailer, and a tree equal to the tree of merging the branches in turn by plain `git merge`; the same with the middle link true-merging gives it two parents, the second its task head; a middle branch conflicting with the first lands in `Conflicted` and the third commit's parent is the first commit; a link whose work `base` already holds lands in `Held`; a task head that names no commit returns an error naming its slug and writes no commit.
`TestMergesClean` rows: the parent and task branch names of a clean pair read true; a branch conflicting with the parent reads false; a ref that does not resolve errors.
`TestMoveCodeTo` rows: a head at `base` moves to the tip and returns true; a head moved past `base` returns false and leaves the head; a tip not descending from `base` errors and moves nothing; tracked dirt is a `*MergeGuardError` whose `WorktreeDirty` answers and moves nothing; a detached head is one whose `DetachedHead` answers.
`TestNewestLandingOf` rows: the newest of two commits naming the slug is found past a commit origin holds; a slug no trailer names is not found; a slug whose only landing lies 501 first-parent commits back, written with `git commit-tree` in a loop, is not found.
