# Card 28 — fabricengine-geometry-reads

**Edit:**
- `internal/fabricengine#Fabric.weftGitDir`
- `internal/fabricengine#resolveGitExcludePath`
- `internal/fabricengine#weftRepoExists`
- `cmd/lyx/checkedcall_test.go`

**Uses:**
- `plan:internal/gitrepo#Repo.GitDir`
- `plan:internal/gitrepo#Repo.CommonDir`

**Intent:** D12: fabricengine's geometry spawns become joins on `gitrepo`'s geometry.
`weftGitDir` returns `f.records.GitDir()`, the per-worktree git dir, which keeps the correspondence index scoped per pair as today.
`resolveGitExcludePath` joins `info/exclude` onto `gitrepo.New(repoDir).CommonDir()`, since the exclude file is per repository and a linked worktree shares it.
`weftRepoExists` reports whether `gitrepo.New(weftRepoRoot).GitDir()` succeeds after its directory check, which is what `rev-parse --is-inside-work-tree` answered; its raw-site marker goes, so `checkedCallPinnedRawSites` for `internal/fabricengine` drops from two to one.
Pure refactor behind unchanged signatures; `index_integration_test.go`, `weftgit_exclude_test.go` and the reconcile tests that drive `weftRepoExists` cover it.

**ImpactSummary:** Three unexported helpers change body; their callers inside fabricengine see the same paths and the same booleans.

**Verify:** lyx gate test --tags integration ./internal/fabricengine -- -run 'TestWeftGitDir|TestIndex|TestWeftGit|TestReconcile'
