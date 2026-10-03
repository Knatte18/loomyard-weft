# Card 2 — gitkit-query-helpers

**Create:**
- `gitkit/query.go`
- `gitkit/query_integration_test.go`

**Edit:**
- `gitkit/doc.go`
- `//CONSTRAINTS.md`
- `//docs/overview.md`

**Uses:**
- `internal/gitkit#MustRun`
- `internal/gitkit#GitStatusPorcelain`

**Intent:**
Give every test package one reachable home for git query and commit plumbing (D4, #325 §1), so external and internal test packages stop each keeping their own copy.

`internal/gitkit/query.go` adds, stdlib-only like `MustRun`, each calling `tb.Helper()` and `tb.Fatalf` on any spawn error or non-zero exit:

- `Git(tb testing.TB, dir string, args ...string) string` — runs `git <args>` in dir and returns trimmed stdout; every other helper below is built on it.
- `RevParse(tb, dir, rev string) string`
- `CurrentBranch(tb, dir string) string`
- `BranchExists(tb, dir, branch string) bool` — answers false for a missing local branch rather than fataling, using `show-ref --verify --quiet refs/heads/<branch>`'s exit status.
- `IsAncestor(tb, dir, ancestor, descendant string) bool` — `merge-base --is-ancestor`, exit 1 answers false, any other non-zero fatals.
- `RevListCount(tb, dir string, args ...string) int`
- `LsFiles(tb, dir string, args ...string) []string`
- `ExcludeLines(tb, dir string) []string` — the non-blank lines of the repo's `info/exclude`, located through `git rev-parse --git-path info/exclude` so a linked worktree resolves to its own common gitdir.
- `CommitFile(tb, dir, rel, content, msg string) string` — writes rel (creating parent directories), stages it alone, commits with msg and returns the new HEAD SHA.
- `CommitFileOnBranch(tb, dir, branch, rel, content, msg string) string` — checks branch out first, then behaves as `CommitFile`.

Signatures are a starting point: where a surveyed private copy shows a variant every caller needs (a bare-repo `RevParse`, extra `LsFiles` args), widen the helper rather than leave the copy behind.

`internal/gitkit/query_integration_test.go` (`//go:build integration`, `package gitkit`) covers each helper against a scratch repo: the true and false answers of `BranchExists` and `IsAncestor`, `RevListCount` over a known range, `ExcludeLines` from a linked worktree, and that `CommitFile` returns the SHA `RevParse(HEAD)` then reports.
`gitkit_test.go`'s existing `TestMain` already installs the hermetic environment for this package.

`gitkit/doc.go` names the new helpers by group rather than restating the export list.
`CONSTRAINTS.md`'s gitkit Leaf Invariant section gains the D6 clause, stated as review discipline: test git plumbing lives in `gitkit`, and a package-local `_test.go` helper that shells out to git is a review flag.
`docs/overview.md`'s `gitkit` sentence stops enumerating primitives and describes the package as the home of test git plumbing.

**ImpactSummary:** Adds exports to gitkit only; no caller changes, and the leaf import allowlist is unchanged.
