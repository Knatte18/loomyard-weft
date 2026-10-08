# Card 10 — gitrepo-reset-keep

**Create:**
- `internal/gitrepo#Repo.ResetKeep`

**Edit:**
- `internal/gitrepo/reset.go`
- `internal/gitrepo/reset_test.go`
- `internal/gitrepo/doc.go`
- `cmd/lyx/gitrepoboundary_test.go`
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `internal/gitrepo#Repo.ResetHard`

**Intent:** Add the keep-reset standalone webster needs (D4) beside the existing hard reset, on the `gitexec` side of the client boundary (`PATTERN-gitrepo-client-boundary`: it mutates the working tree).
`ResetKeep` validates `sha` exactly as `ResetHard` does (`ErrInvalidSHA` before any git spawn) and runs `git reset --keep <sha>` through `runChecked`.
git's keep form moves HEAD and the index to `sha` and carries every uncommitted change across, refusing, with nothing changed, when an uncommitted change sits in a path that differs between HEAD and `sha`; that refusal comes back as the checked `*gitexec.GitError`, wrapped with the method's own context, so a caller can surface git's message.
`reset_test.go` (integration tier, real repositories) gains `TestResetKeep`: a move back over a committed change with an unrelated dirty file kept, a refusal over a dirty file the move would overwrite that leaves HEAD and the file untouched, and the invalid-SHA rejection; `test-budget.yaml` raises `internal/gitrepo` by one for it.
`gitrepoPinnedRunBoundMethods` in `cmd/lyx/gitrepoboundary_test.go` gains `ResetKeep` beside `ResetHard`, so `TestGitrepoBoundary_PinnedRunCallSites` pins the new `runChecked` call site; `cmd/lyx/checkedcall_test.go` needs no edit, since `runChecked` is not a raw site.
The gitrepo package doc lists `ResetKeep` among the CLI-bound methods beside `ResetHard`.

**ImpactSummary:** Adds one method to gitrepo.Repo and its entry in the pinned run-bound method set; no existing caller changes, and its one consumer is card 15's standalone reset.
