```yaml
lens: code-tests
outcome: reported
```

**Per-behaviour list**

1. `ResolveWithin` accepts the anchored directory and a directory below it. `TestResolveWithin` rows `AnchoredDirectory` and `BelowTheAnchoredDirectory` pin it (`lymcwd_test.go`). Covered.
2. `ResolveWithin` maps a cwd in the weft sibling to the owner's `Location`. Rows `SiblingRecordsDirectory`, `SiblingRoot` and `BelowTheAnchorsRecordsLink` pin it, each compared against `want`. Covered.
3. `ResolveWithin` refuses a cwd outside the anchor and names the anchored directory. Rows `SiblingOfTheAnchor` and `WorktreeRootAboveTheAnchor` pin it. Covered.
4. `ResolveWithin` returns `ErrNotAGitRepo` outside a git repo. None. The only `ErrNotAGitRepo` test, `TestResolve_NotAGitRepo`, calls `Resolve`. It shares `gitWorktreeRoot` with `ResolveWithin`, but nothing fails if `ResolveWithin`'s own call is broken.
5. `ownerOfRecordsSibling` treats a worktree whose name ends in `-weft` as itself when no owner directory sits beside it. None. Every `TestResolveWithin` sibling row has its owner present. A broken `os.Stat` check, or removing the check, passes all rows. The summary names this as a design decision.
6. `ResolveWithin` with no recorded marker falls back to `.`. None. The fixture always records `backend`. `anchorRelOrRoot` has no dedicated test, but `Resolve`'s marker-absent test covers `resolveCore`, which now uses it.
7. `ErrCwdOutsideAnchor`'s message changed to "cwd is not at the recorded fabric anchor". No test asserts the text. Callers match with `errors.Is` (`lymcwd_test.go:48`, `gate_test.go:82`). Nothing was left unmatched, so this is not a gap.
8. `pathWithin` is a new helper. `TestResolveWithin` reaches it only through the public `ResolveWithin`. Under this repo's public-surface testing rule, a per-helper test is not wanted. A `..`-prefixed directory name is untested (for example a child named `..foo` under the anchor, which `HasPrefix(rel, "..")` is built to admit). Windows case folding is untested. Both are LOW coverage gaps, not blocking.
9. The shuttle pre-run swaps to `ResolveWithin` for `await-forks` only. None. `TestAwaitForksVerb` and the registration rows in `internal/shuttlecli/cli_test.go` never run the verb from a weft cwd. A regression in the `cmd.Name() == awaitForksVerb` dispatch, such as choosing `Resolve` or swapping every verb, fails no test. This is the behaviour the issue is about, so it is the main gap.
10. `NamedWorktree` and `resolveCore` now share `anchorRelOrRoot`. Existing `Resolve` and `NamedWorktree` tests cover this refactor. Covered.
11. `TestResolveWithin` against existing tests: it is a distinct behaviour and does not duplicate `Resolve`'s rows. No redundancy.

**Findings**

- MEDIUM, scope: no test runs `await-forks` from a weft cwd or checks that only `await-forks` takes `ResolveWithin`. Location: `internal/shuttlecli/cli.go:87-91`, tests in `internal/shuttlecli/cli_test.go` (`TestAwaitForksVerb` at line 418). Fix: add a row that runs `await-forks` from a weft-sibling cwd and expects it to pass the resolution gate. A second row should show another verb such as `state` still refusing a non-anchor cwd.
- LOW, scope: a worktree named `<x>-weft` with no `<x>` beside it is untested, although `ownerOfRecordsSibling` branches on it. Location: `internal/lymcwd/lymcwd.go:169-178`, `lymcwd_test.go:TestResolveWithin`. Fix: add a row with a checkout whose own name ends in `-weft` and has no owner, expecting it to resolve as itself, with the strict anchor check applying.
- LOW, scope: `ResolveWithin` returning `ErrNotAGitRepo` is untested. Location: `internal/lymcwd/lymcwd.go:144-147`. Fix: add a row, or a short test, that calls `ResolveWithin` on a non-git temp directory.
- LOW, scope: a no-marker anchor (`.` fallback) is untested for `ResolveWithin`. Location: `internal/lymcwd/lymcwd.go:151`. Fix: add a case with a checkout that has no marker, resolving from a subdirectory.

**Tool calls**
1. One grep over `internal/**/*_test.go` for the new symbols and for await-forks tests. My context held `lymcwd_test.go` only as a diff and had no shuttlecli tests.
