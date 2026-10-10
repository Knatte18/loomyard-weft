# Card 5 — gitrepo-bolt-primitives

**Edit:**
- `internal/gitrepo#Repo.pushWithRebaseRetry`
- `internal/gitrepo/push_test.go`
- `internal/gitrepo/doc.go`
- `//cmd/lyx/gitrepoboundary_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `internal/gitrepo#ErrNoUpstream`
- `internal/gitrepo#Repo.UpstreamSHA`
- `internal/gitrepo#CommitDetail`
- `internal/gitrepo#Repo.CommitDetail`
- `internal/gitrepo#Repo.CherryPick`
- `internal/gitrepo#Repo.CherryPickAbort`
- `internal/gitrepo#ErrPullRebaseFailed`
- `internal/gitrepo/replay_integration_test.go`

**Intent:** The git primitives D4 and D5 need, split by `PATTERN-gitrepo-client-boundary`.
Two go-git reads through `readGoGit`: `UpstreamSHA` resolves the current branch's configured upstream (`branch.<name>.remote` and `.merge`) to its remote-tracking ref's SHA, returning `ErrNoUpstream` bare when the branch has none; `CommitDetail` returns one commit's full message and the repo-relative paths its tree changes against its first parent, every path for a root commit, so a caller can classify a commit without spawning git.
Two tree-mutating `runChecked` calls: `CherryPick` runs `git cherry-pick --empty=drop <sha>` with the SHA validated first (`ErrInvalidSHA`), so a commit whose change the branch already carries is dropped with exit 0 as `pull --rebase` drops it, and returns the wrapped `*gitexec.GitError` on a conflict; `CherryPickAbort` runs `git cherry-pick --abort` and treats git's no-cherry-pick-in-progress exit as success, mirroring the rebase abort in `pushWithRebaseRetry`.
Both join `gitrepoPinnedRunBoundMethods` in `cmd/lyx/gitrepoboundary_test.go`.
`pushWithRebaseRetry` wraps the error it returns after a rebase it aborted with `ErrPullRebaseFailed`, so `fabricengine`'s Bolt push can tell a replayable conflict from every other push failure with `errors.Is`; the message text is unchanged.
`doc.go`'s method lists name the four new methods on their sides of the boundary.
Tests: `TestPush` gains a row asserting the rebase-conflict error satisfies `errors.Is(err, ErrPullRebaseFailed)` and a plain push failure does not; the new `integration` file holds `TestUpstreamSHA` (tracking branch, no upstream, upstream moved after a fetch), `TestCommitDetail` (message, changed paths, root commit, invalid SHA) and `TestCherryPick` (clean pick lands the commit, a pick whose change the branch already carries succeeds with HEAD unmoved, conflicting pick returns the git error and `CherryPickAbort` restores HEAD, abort with nothing in progress succeeds); the budget row for `internal/gitrepo` rises by three.

**ImpactSummary:** One private push helper gains an error wrapper that no existing caller inspects; the new methods have no callers until card 7, the sentinel none until card 13, and the pinned boundary list grows by the two CLI-bound ones.
