# Card 17 — gitoracle-package

**Create:**
- `gitrepo/internal/gitoracle/oracle.go`
- `gitrepo/internal/gitoracle/leaf_enforcement_test.go`

**Edit:**
- `gitrepo/gogit_test.go`
- `gitrepo/parity_test.go`
- `//cmd/lyx/checkedcall_test.go`
- `//CONSTRAINTS.md`

**Delete:**
- `gitrepo/oracle_test.go`

**Intent:**
Give gitrepo's parity oracle one home both test packages can reach, so `gogit_test.go` stops carrying a verbatim copy (#325 §6).

`internal/gitrepo/internal/gitoracle/oracle.go` holds the four oracle functions and their no-commits sentinel, moved from `oracle_test.go` and exported (`CurrentSHA`, `SHAExists`, `ChangedFilesSince`, `CurrentBranch`, `ErrNoCommits`), each taking `testing.TB` and an explicit worktree directory as today.
Their bodies keep their parsing conventions verbatim: the oracle is D9-protected, restructured only, and its independence from `gitrepo` is its whole purpose; `oracle_test.go`'s "why an oracle, not a copy" explanation becomes the package doc.
It imports stdlib and `internal/gitexec` only, never `internal/gitrepo` or `internal/gitkit`.
Each raw `gitexec.RunGit` call keeps the raw form (the oracle reads exit codes and stderr) and carries a `//gitexec:raw` marker saying so; `cmd/lyx/checkedcall_test.go`'s `checkedCallPinnedRawSites` gains one row for `internal/gitrepo/internal/gitoracle` pinning exactly that count.
That row is the only change to the guard: its token set, floor and other rows stay verbatim, and planting one unmarked `gitexec.RunGit` in the oracle must fail it (remove the plant).

`leaf_enforcement_test.go` is the import-allowlist row enforcing independence, modelled on `internal/gitkit/leaf_enforcement_test.go`: production files in `gitoracle` import only stdlib and `internal/gitexec`.
Plant a `gitrepo` import, watch it fail, remove it.

`parity_test.go` (package `gitrepo_test`) and `gogit_test.go` (package `gitrepo`) call `gitoracle`; `gogit_test.go`'s verbatim copy and its package-boundary justification go.
`newStandaloneRepo` in `gogit_test.go` duplicates `gitrepo_test.go`'s `newRepo` across the same boundary: after card 6 its git setup is gitkit calls, so keep it only if its tests need package-internal access, otherwise move those tests to `gitrepo_test` and use `newRepo`.

`CONSTRAINTS.md`'s gitrepo Client Boundary Invariant gains one line: the parity oracle `internal/gitrepo/internal/gitoracle` imports only stdlib and `gitexec`, never `gitrepo`, enforced by its leaf test.

What this admits: a production (non-test) package whose callers are only tests, carrying raw git calls; the pinned row and the leaf test bound it to the oracle's reads.

**ImpactSummary:** gitrepo integration tests switch to the moved oracle; the checked-call guard gains one pinned row; no gitrepo production code changes.
