# Card 2 — gitkit-query-helpers

**Create:**
- `gitkit/query.go`
- `gitkit/query_integration_test.go`

**Edit:**
- `gitkit/gitkit.go`
- `gitkit/doc.go`
- `//CONSTRAINTS.md`
- `//docs/overview.md`

**Uses:**
- `internal/gitkit#MustRun`
- `internal/gitkit#GitStatusPorcelain`

**Intent:**
Give every test package one reachable home for git query and commit plumbing (D4, #325 §1), so external and internal test packages stop each keeping their own copy.

The one new spawning primitive lives in `gitkit.go`, beside `MustRun`: `cmd/lyx/spawnobservability_test.go` allowlists `internal/gitkit/gitkit.go` by path, and gitkit cannot import `internal/logger` under its Leaf Invariant, so an `exec.Command` in any other gitkit file would fail that guard.
`gitkit.go` gains `Git(tb testing.TB, dir string, args ...string) string`, which runs `git <args>` in dir and returns trimmed stdout, and an unexported sibling returning stdout and the exit code without fataling on a non-zero exit, for the two boolean helpers.
`internal/gitkit/query.go` imports no `os/exec` and adds, stdlib-only, each built on those two and calling `tb.Helper()` and `tb.Fatalf` on any spawn error or unexpected non-zero exit:

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

`internal/gitkit/query_integration_test.go` (`//go:build integration`, `package gitkit`) covers `Git` and each helper against a scratch repo: the true and false answers of `BranchExists` and `IsAncestor`, `RevListCount` over a known range, `ExcludeLines` from a linked worktree, and that `CommitFile` returns the SHA `RevParse(HEAD)` then reports.
`gitkit_test.go`'s existing `TestMain` already installs the hermetic environment for this package.

`gitkit/doc.go` names the new helpers by group rather than restating the export list.
`CONSTRAINTS.md`'s gitkit Leaf Invariant section gains the D6 clause, stated as review discipline: test git plumbing lives in `gitkit`, and a package-local `_test.go` helper that shells out to git is a review flag.
`docs/overview.md`'s `gitkit` sentence stops enumerating primitives and describes the package as the home of test git plumbing.

**ImpactSummary:** Adds exports to gitkit only; no caller changes, and neither the leaf import allowlist nor the spawn-observability allowlist changes, since every new spawn sits in the already-allowlisted `gitkit.go`.
