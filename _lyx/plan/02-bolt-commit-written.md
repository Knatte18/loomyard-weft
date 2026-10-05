# Card 2 — Bolt: write and commit named paths under the board write lock

**Create:**
- `internal/fabricengine#Bolt.CommitWritten`

**Edit:**
- `internal/fabricengine/bolt.go`
- `internal/fabricengine/bolt_integration_test.go`

**Uses:**
- `internal/fabricengine#Bolt.Commit`
- `internal/fabricengine#ScopedPathspec`
- `internal/fabricengine/stencilcommit.go`

**Intent:**
Add `Bolt.CommitWritten`, the route D4 and D5 commit hub-wide config through.
It acquires the board write lock at `filepath.Join(b.path, BoardWriteLockFile)` (the same file `boardengine`'s `commitDirty` and `CommitSeededStencils` take), calls `write`, and commits only the repo-relative paths `write` returns, with a scoped pathspec through `gitrepo`'s `StageAndCommit`, the way `CommitSeededStencils` does.
The lock is released only after the commit, so a concurrent board sync (whose `Bolt.Commit` stages everything) can never commit a half-written file.

Contract:
- A `write` error is returned unwrapped, with no commit made, so a caller can tell a refused write from a failed commit.
- An empty path list commits nothing and returns `committed == false`.
- `opts.SkipGit` still runs `write` under the lock but skips the commit, so the `WEFT_SKIP_GIT` test gate keeps working.
- A deleted path in the list stages its deletion.
- It never pushes; callers push with `Bolt.Push` afterwards, outside the write lock.

Bound: the commit covers only the paths `write` names, so unrelated dirty files in `_board` stay for the next board sync.
Update `bolt.go`'s header comment, which names `Commit`/`Push`/`Sync` as the handle's surface.

Coverage, in `bolt_integration_test.go` (integration tier, real git, like the neighbouring Bolt tests):
- a dirty repo holding two changed files, where `write` touches one, commits exactly that one and leaves the other dirty;
- a `write` error returns that error and leaves HEAD unchanged.

**ImpactSummary:** One new method on `Bolt`; existing `Commit`/`Push`/`Sync` are untouched.

**Commit:** 2: fabricengine: Bolt.CommitWritten commits named paths under the board write lock
