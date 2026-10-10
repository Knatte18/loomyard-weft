# Card 7 — bolt-pull-first-seed-drop

**Edit:**
- `internal/fabricengine#Bolt`
- `internal/fabricengine#Bolt.Push`
- `internal/fabricengine#resolvePathOwnership`
- `internal/fabricengine/bolt_integration_test.go`
- `internal/fabricengine/doc.go`
- `//cmd/lyx/destructiveguard_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/fabricengine#BoltSkip` -> `type BoltSkip string`
- `plan:internal/fabricengine#BoltSkipFetchFailed` -> `const BoltSkipFetchFailed BoltSkip = "fetch_failed"`
- `plan:internal/fabricengine#BoltSkipDiverged` -> `const BoltSkipDiverged BoltSkip = "diverged"`
- `plan:internal/fabricengine#BoltWriteResult` -> `type BoltWriteResult struct`
- `plan:internal/fabricengine#Bolt.PullThenCommitWritten` -> `func (b *Bolt) PullThenCommitWritten(message string, write func() ([]string, error), rec *Mutations) (BoltWriteResult, error)`
- `plan:internal/fabricengine#Bolt.pullFirst` -> `func (b *Bolt) pullFirst(rec *Mutations) (BoltSkip, string, error)`
- `plan:internal/fabricengine#seedCommitsAhead` -> `func seedCommitsAhead(repo *gitrepo.Repo, ahead []string) (seed, other []string, err error)`
- `plan:internal/fabricengine#pathOwnershipBoardWorktree` -> `const pathOwnershipBoardWorktree pathOwnershipKind = 10`
- `plan:internal/fabricengine#ownedBoardWorktree` -> `func ownedBoardWorktree() pathOwnership`
- `plan:internal/fabricengine#dropSeedCommits` -> `func dropSeedCommits(rec *Mutations, req pathRequest, repo *gitrepo.Repo, upstream string, ahead []string) error`
- `plan:internal/fabricengine#KindCommitsDropped` -> `const KindCommitsDropped Kind = "commits_dropped"`

**Uses:**
- `plan:internal/gitrepo#Repo.UpstreamSHA`
- `plan:internal/gitrepo#Repo.CommitDetail`
- `plan:internal/gitrepo#Repo.CherryPick`
- `plan:internal/gitrepo#Repo.CherryPickAbort`
- `plan:internal/gitrepo#ErrPullRebaseFailed`
- `plan:internal/gitrepo#ErrNoUpstream`
- `internal/gitrepo#Repo.CommitsNotIn`
- `internal/gitrepo#Repo.Fetch`
- `internal/gitrepo#Repo.MergeFFOnly`
- `internal/gitrepo#Repo.ResetKeep`
- `plan:internal/fabricengine#IsSeedCommit`
- `internal/fabricengine#checkPathRequest`
- `internal/fabricengine#isAnyWorktreeOf`
- `internal/fabricengine#BoardDirName`
- `internal/fabricengine#KindRepoAdvanced`

**Intent:** D4's pull-first helper on `Bolt` and D5's drop primitive behind the destruction gate.
`dropSeedCommits` lives in `destroy.go`: it runs `checkPathRequest` over `req`, then `ResetKeep` to `upstream`, then replays every non-seed commit of `ahead`, oldest first, through `CherryPick`; on a replay failure it runs `CherryPickAbort`, `ResetKeep` back to the tip it started from and returns the failure, so Bolt is never left mid-replay.
It records `KindCommitsDropped` with the dropped SHAs in the detail once the reset observably moved HEAD, and nothing on a refusal; it is appended to `destructiveGuardRecordingExecutors`, `.CherryPick(` and `.ResetKeep(` join `destructiveGuardBannedTokens` so no other `fabricengine` file reaches them, and `destroy.go`'s header comment lists the new primitive.
The request is fixed by `pullFirst`: container the parent of the board dir, target the board dir, ownership `ownedBoardWorktree`, a new kind that holds when the target's base name is `BoardDirName` and it is a worktree of its own repo (`isAnyWorktreeOf(target, target)`), dirtiness `dirtyScopeTracked`, force always false.
`seedCommitsAhead` splits an ahead set with `CommitDetail` and `IsSeedCommit`.
`pullFirst` runs with the board write lock held: `Fetch`, then `UpstreamSHA` (no upstream is `BoltSkipDiverged`-free: an unborn or untracked Bolt returns no skip and writes as today), then `CommitsNotIn(HEAD, upstream)`; when the ahead set holds seed commits it calls `dropSeedCommits`; a Bolt then behind and not ahead is fast-forwarded with `MergeFFOnly` and records `KindRepoAdvanced`; a Bolt still ahead over a moved upstream, a refused or conflicted drop, returns `BoltSkipDiverged` with a detail naming the board dir, `git pull --rebase` in it and `lyx board sync` as the way forward; a `Fetch` error returns `BoltSkipFetchFailed` with a detail naming the remote and a re-run of the command.
Both skips are logged at Warn once, here, so no caller repeats the text, and neither names a denied command (D6).
`PullThenCommitWritten` is `CommitWritten`'s pull-first sibling, not a re-sign, because `configcli`'s hub-wide edit must keep working offline: it takes the board write lock, calls `pullFirst`, returns the skip without calling `write` when one is reported, and otherwise runs `write`, commits the returned board-relative paths through the scoped pathspec as `CommitWritten` does, and records `KindFileWritten` per path and `KindCommitCreated` on a landed commit; `BoltWriteResult` embeds `MutationRecord` and carries `SHA`, `Committed`, `Skipped BoltSkip` and `SkipDetail`, and joins `destructiveGuardMutatingResultTypes`.
It never pushes.
`Bolt.Push` keeps delegating to `pushWeftAt` and, when the error satisfies `ErrPullRebaseFailed`, takes the board write lock, runs `seedCommitsAhead` and `dropSeedCommits` when seed commits are ahead, releases the lock and retries `pushWeftAt` once; with no seed commits ahead, or when the drop refuses, the original error is returned as today.
`doc.go` documents the pull-first rule, the skip outcomes, the drop primitive's bound (only commits `IsSeedCommit` admits are dropped; a non-seed commit is replayed or, on conflict, kept at its original tip) and the push retry.
Tests in `bolt_integration_test.go` against its plain-repo-plus-bare-remote fixture, with a second clone of the bare moving the upstream: a new `TestBolt_PullThenCommitWritten` with subtests for behind-only fast-forward then commit, seed commits ahead dropped and the write rerun on the pulled copy, non-seed commits over a moved upstream skipped diverged with nothing written and the Warn text captured, fetch failure skipped with nothing written, a replay conflict restoring the original tip, a tracked-dirty board refused and restored, and a table of commits the drop refuses to classify as seed: bare `lyx: seed stencils`, `lyx: sync stencils (…)`, a hub-wide config message, and a labelled seed message touching a path outside the subtrees; a new `TestBolt_Push_DropsSeedCommitsOnRebaseConflict` with a seed and a board commit ahead of a moved upstream landing the board commit after the drop, and a conflicting board commit leaving Bolt at its tip with the error returned; the budget row for `internal/fabricengine` rises by two.
Existing `CommitWritten` subtests are unchanged.

**ImpactSummary:** `Bolt.Push` gains a retry path reached only on a rebase conflict with seed commits ahead, so every board push caller keeps its contract otherwise; `resolvePathOwnership` gains one case; the new verb has no callers until cards 8, 10 and 11.
