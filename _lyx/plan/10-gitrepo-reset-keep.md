# Card 10 — gitrepo-reset-keep

**Create:**
- `plan:internal/gitrepo#Repo.ResetKeep` -> `func (r *Repo) ResetKeep(sha string) error`
- `plan:internal/websterengine#resetKeep` -> `func resetKeep(worktree, sha string) error`

**Edit:**
- `internal/gitrepo/reset.go`
- `internal/gitrepo/reset_test.go`
- `internal/gitrepo/doc.go`

**Uses:**
- `internal/gitrepo#Repo.ResetHard`

**Intent:** Add the keep-reset standalone webster needs (D4) beside the existing hard reset, on the `gitexec` side of the client boundary (`PATTERN-gitrepo-client-boundary`: it mutates the working tree), and webster's one wrapper over it.
`ResetKeep` validates `sha` exactly as `ResetHard` does (`ErrInvalidSHA` before any git spawn) and runs `git reset --keep <sha>` through `runChecked`.
git's keep form moves HEAD and the index to `sha` and carries every uncommitted change across, refusing, with nothing changed, when an uncommitted change sits in a path that differs between HEAD and `sha`; that refusal comes back as the checked `*gitexec.GitError`, wrapped with the method's own context, so a caller can surface git's message.
`resetKeep` lives in `internal/websterengine/gitwrap.go`, the one webster file that runs git, beside its read-only helpers: it calls `gitrepo.New(worktree).ResetKeep(sha)` and wraps the error with webster's context, and its doc comment says it is webster's one mutating git helper, used only by the standalone reset (card 12).
`reset_test.go` (integration tier, real repositories) gains `TestResetKeep`: a move back over a committed change with an unrelated dirty file kept, a refusal over a dirty file the move would overwrite that leaves HEAD and the file untouched, and the invalid-SHA rejection.
The gitrepo package doc lists `ResetKeep` among the CLI-bound methods beside `ResetHard`.

**ImpactSummary:** Adds one method to gitrepo.Repo and one unexported webster wrapper; no existing caller changes.
