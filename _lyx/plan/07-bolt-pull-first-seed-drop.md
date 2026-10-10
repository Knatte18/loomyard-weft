# Card 7 — bolt-pull-first-seed-drop

**Edit:**
- `internal/fabricengine#resolvePathOwnership`
- `internal/fabricengine#checkPathDirtiness`
- `internal/fabricengine/doc.go`
- `internal/fabricengine/livestate_mutationoracle_test.go`
- `internal/fabricengine/export_test.go`
- `//cmd/lyx/destructiveguard_test.go`
- `//cmd/lyx/testdata/test-budget.yaml`

**Create:**
- `plan:internal/fabricengine#BoltSkip` -> `type BoltSkip string`
- `plan:internal/fabricengine#BoltSkipFetchFailed` -> `const BoltSkipFetchFailed BoltSkip = "fetch_failed"`
- `plan:internal/fabricengine#BoltSkipDiverged` -> `const BoltSkipDiverged BoltSkip = "diverged"`
- `plan:internal/fabricengine#BoltSkipDirty` -> `const BoltSkipDirty BoltSkip = "dirty"`
- `plan:internal/fabricengine#BoltWrite` -> `type BoltWrite struct`
- `plan:internal/fabricengine#BoltWriteResult` -> `type BoltWriteResult struct`
- `plan:internal/fabricengine#Bolt.PullThenCommitWritten` -> `func (b *Bolt) PullThenCommitWritten(writes []BoltWrite, rec *Mutations) (BoltWriteResult, error)`
- `plan:internal/fabricengine#Bolt.pullFirst` -> `func (b *Bolt) pullFirst(rec *Mutations) (BoltSkip, string, error)`
- `plan:internal/fabricengine#seedCommitsAhead` -> `func seedCommitsAhead(repo *gitrepo.Repo, ahead []string) (seed, other, touched []string, err error)`
- `plan:internal/fabricengine#boardDropRequest` -> `func boardDropRequest(boardDir string, touched []string) pathRequest`
- `plan:internal/fabricengine#pathOwnershipBoardWorktree` -> `const pathOwnershipBoardWorktree pathOwnershipKind = 10`
- `plan:internal/fabricengine#ownedBoardWorktree` -> `func ownedBoardWorktree() pathOwnership`
- `plan:internal/fabricengine#pathDirtinessTrackedOn` -> `const pathDirtinessTrackedOn pathDirtinessKind = 4`
- `plan:internal/fabricengine#dirtyTrackedOn` -> `func dirtyTrackedOn(paths []string) pathDirtiness`
- `plan:internal/fabricengine#dropSeedCommits` -> `func dropSeedCommits(rec *Mutations, req pathRequest, repo *gitrepo.Repo, upstream string, ahead []string) error`
- `plan:internal/fabricengine#KindCommitsDropped` -> `const KindCommitsDropped Kind = "commits_dropped"`
- `internal/fabricengine/boltpullfirst_integration_test.go`

**Uses:**
- `plan:internal/gitrepo#Repo.UpstreamSHA`
- `plan:internal/gitrepo#Repo.CommitDetail`
- `plan:internal/gitrepo#Repo.CherryPick`
- `plan:internal/gitrepo#Repo.CherryPickAbort`
- `plan:internal/gitrepo#ErrNoUpstream`
- `internal/gitrepo#PushLockFileName`
- `internal/gitrepo#Repo.CommitsNotIn`
- `internal/gitrepo#Repo.Fetch`
- `internal/gitrepo#Repo.MergeFFOnly`
- `internal/gitrepo#Repo.ResetKeep`
- `plan:internal/fabricengine#IsSeedCommit`
- `internal/fabricengine#checkPathRequest`
- `internal/fabricengine#isRegisteredLinkedWorktreeIn`
- `internal/fabricengine#WeftWarpSlug`
- `internal/fabricengine#dirtyPathsOutside`
- `internal/fabricengine#BoardDirName`
- `internal/fabricengine#KindRepoAdvanced`

**Intent:** D4's pull-first helper on `Bolt` and D5's drop primitive behind the destruction gate; the push-path retry that reuses the primitive is card 13.
`dropSeedCommits` lives in `destroy.go`: it runs `checkPathRequest` over `req`, then `ResetKeep` to `upstream`, then replays every non-seed commit of `ahead`, oldest first, through `CherryPick`, which drops a commit whose change upstream already carries as `pull --rebase` does; on a replay failure it runs `CherryPickAbort`, `ResetKeep` back to the tip it started from and returns the failure, so Bolt is never left mid-replay.
It records `KindCommitsDropped` with the dropped SHAs in the detail once the reset observably moved HEAD, and nothing on a refusal; it is appended to `destructiveGuardRecordingExecutors`, `.CherryPick(` and `.ResetKeep(` join `destructiveGuardBannedTokens` so no other `fabricengine` file reaches them, `KindCommitsDropped` joins `manifestObservableKind` as a git-state kind (`false`, like `KindWorktreeReset`), and `destroy.go`'s header comment lists the new primitive.
`boardDropRequest`, also in `destroy.go`, is the one request both drop callers build: container the board dir's parent, target the board dir, ownership `ownedBoardWorktree`, dirtiness `dirtyTrackedOn(touched)`, force always false.
`ownedBoardWorktree`'s kind holds only when the target's base name is `BoardDirName` and some directory directly in the target's parent whose name `WeftWarpSlug` admits registers the target as a linked worktree (`isRegisteredLinkedWorktreeIn`); that is the bound on the new edge: the drop can reset only a hub's `_board` that a weft repo of the same hub owns, never another directory named `_board`, and never with force.
`dirtyTrackedOn(paths)` is a new dirtiness kind that refuses only on a dirty tracked path in `paths`, read through `dirtyPathsOutside`'s porcelain parsing; `touched` is every path the ahead commits change, so a pending board write on any other path, the normal state between a board write and its sync, is carried across: `ResetKeep` refuses with nothing changed when a dirty path differs between HEAD and upstream, and the restore to the original tip then crosses only paths that check already admitted.
`seedCommitsAhead` splits an ahead set with `CommitDetail` and `IsSeedCommit` and returns the union of their changed paths as `touched`.
`pullFirst` runs with the board write lock held and, for its whole run, also holds `gitrepo.PushLockFileName` in the board dir, taken after the board write lock and released before it returns: `PushCoalesced` holds that lock across its `pull --rebase` and abort and never takes the board write lock, so without it another process's push could rebase `_board` while the drop resets and replays it or the fast-forward moves it; nothing holds the push lock while waiting for the board write lock, so the order cannot deadlock.
Under both locks it runs `Fetch`, then `UpstreamSHA` (no upstream returns no skip: an unborn or untracked Bolt writes as today), then `CommitsNotIn(HEAD, upstream)`; when the ahead set holds seed commits it calls `dropSeedCommits` with `boardDropRequest`; a Bolt then behind and not ahead is fast-forwarded with `MergeFFOnly` and records `KindRepoAdvanced`.
Its skips: a `Fetch` error returns `BoltSkipFetchFailed` with a detail naming the remote and a re-run of the command; a dirtiness refusal, or a `ResetKeep` or `MergeFFOnly` that refuses over a local change, returns `BoltSkipDirty` with a detail naming the dirty paths, the next `lyx board sync` and a re-run; a Bolt still ahead over a moved upstream, or a drop refused on containment or ownership or whose replay conflicted, returns `BoltSkipDiverged` with a detail naming the board dir, `git pull --rebase` in it and `lyx board sync`.
Each skip is logged at Warn once, here, so no caller repeats the text, and none names a denied command (D6).
A drop is logged at Info here with the dropped SHAs, beside its `KindCommitsDropped` entry in `rec`, so a caller whose record reaches no envelope still sees the reset in the log.
`PullThenCommitWritten` is `CommitWritten`'s pull-first sibling, not a re-sign, because `configcli`'s hub-wide edit must keep working offline.
It takes the board write lock once, calls `pullFirst` once, and returns the skip without running any write when one is reported; otherwise it runs each `BoltWrite` in order, a `Message` and a `Write` returning board-relative paths, and commits each write's paths through the scoped pathspec as `CommitWritten` does, an empty list committing nothing, so several subtrees land as separate commits behind one pull and no later write's pull can drop an earlier write's commit.
A write or commit error stops the sequence and is returned, keeping the commits already landed.
It records `KindFileWritten` per path and `KindCommitCreated` per landed commit; `BoltWriteResult` embeds `MutationRecord` and carries `SHAs` (landed commits in order), `Committed`, `Skipped BoltSkip` and `SkipDetail`, and joins `destructiveGuardMutatingResultTypes`.
It never pushes.
`doc.go` documents the pull-first rule, the three skip outcomes, the lock order (board write lock, then the push lock, across every reset, replay and fast-forward of Bolt) and the drop primitive's bound (only commits `IsSeedCommit` admits are dropped, only in a hub's own `_board`; a non-seed commit is replayed or, on conflict, kept at its original tip).
Tests in the new `package fabricengine_test` integration file, over a `hubforge` hub whose upstream a second clone of `RecordsBare` moves: `TestBolt_PullThenCommitWritten` with subtests for behind-only fast-forward then commit, two writes landing as two commits behind one pull, seed commits ahead dropped and the write rerun on the pulled copy, non-seed commits over a moved upstream skipped diverged with nothing written and the Warn text captured, fetch failure skipped with nothing written, a replay conflict restoring the original tip, a dirty board file outside the touched paths carried across the drop, a dirty file on a touched path skipped dirty with Bolt unmoved, a labelled seed message touching a path outside the subtrees kept and replayed rather than dropped, and a push lock held by the test, taken on `gitrepo.PushLockFileName` in the board dir before the call, under which Bolt's HEAD is still the original tip, the drop and the write landing only after the test releases it; `TestBoardDropRequest_RefusesForeignBoard` builds its foreign `_board` as a linked worktree of the same `hubforge` hub's records repo, added with `gitkit.MustRun` and `git worktree add` at `<t.TempDir()>/elsewhere/_board`, so the target is a real registered `_board` whose parent holds no weft sibling, gives it to `dropSeedCommits` through `boardDropRequest` and asserts the ownership refusal with nothing recorded;
`export_test.go` re-exports `dropSeedCommits` and `boardDropRequest` for that external-package test, as it already does for the gate's other executors; the budget row for `internal/fabricengine` rises by two.
Existing `CommitWritten` subtests are unchanged.

**ImpactSummary:** Adds one ownership kind and one dirtiness kind to the gate, both reached only by the new primitive, so every existing destroy call site keeps its checks; the new verb has no callers until cards 8, 10 and 11.
