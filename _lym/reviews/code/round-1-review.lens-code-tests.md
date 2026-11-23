```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, find the test row that fails if it breaks.

**Per behaviour, the test that pins it**

1. **`Lint` against a told base reads the working tree, committed and uncommitted changes together.**
   - `TestLint_OverARepository` / "the working tree form against a base sees committed and uncommitted changes" expects `added.go:3`, `renamed.go:7`, `untracked.go:3` (`internal/commentlint/lint_integration_test.go`, step after line 90).
   - `TestLintCommentsCmd_OverARepository` / "lint-comments --base lints uncommitted edits too" expects literal `edited.go:3` and `wrapped.go:3` envelope lines.
   - Both fail on the merge base, which reads HEAD as the new side.
2. **An empty base means HEAD, working tree and untracked files included.**
   - `TestLint_OverARepository` / "the working tree form against HEAD sees an edit and an untracked file but not legacy lines" expects `renamed.go:7`, `untracked.go:3`.
3. **Untracked `.go` files are added when a non-HEAD base is told.**
   - `untracked.go:3` in the base-form subtest (behaviour 1). Before the diff, the untracked addition ran only for an empty base.
4. **An untracked file that the base holds gets its own path as the base side, so only its new lines count.**
   - `TestLint_OverARepository` / "an untracked file the base holds flags only its new lines" (lint_integration_test.go:100–114).
   - The `indexless.go` content equals the base copy. Leaving `basePath` empty would flag its legacy wrapped lines, so the literal want `renamed.go:7`, `untracked.go:3` fails.
5. **`LintCommitRange` reads HEAD, never the working tree.**
   - `TestLint_OverARepository` / "the commit range form sees committed changes but not uncommitted ones" expects only `added.go:3`, with an uncommitted edit and an untracked file present.
6. **A pure rename flags nothing, in both forms.**
   - `TestLint_OverARepository` / "a pure rename flags nothing" now loops over `Lint` and `LintCommitRange`.
7. **The verify gate calls `LintCommitRange`.**
   - `TestVerifyGate_RoundScenario` / "a first round lints from the merge base and fails on a fixed-column comment the round added" writes an untracked wrapped file and asserts no finding names `untracked.go` (`internal/loomshed/verifygate_integration_test.go`, around line 245).
   - Swapping back to `Lint` would report it.
8. **Non-`.go` untracked paths skip the `cat-file` probe.**
   - None.
   - This is an internal optimisation with no observable output: the later `.go` filter dropped those paths before too.
   - No row can tell it apart, so this is no finding.
9. **The `cat-file` failure path (`commentlint: look for %s at %s`).**
   - None.
   - `RunGit` returns an error only when git cannot run, and the `git diff` call just before it already covers that case.
   - Not reasonably reachable, so this is no finding.
10. **The `--base` flag description and the `Long` help, `lintcomments.go:70` (the mutation survivor).**
    - None, and none is needed. This is static prose, which the rubric exempts; asserting it would itself be a test-economy finding.
11. **The empty-base callers (`gatecli` `runLints`, `lint-comments` without `--base`) behave as before.**
    - `TestLint_OverARepository`'s empty-base subtest (behaviour 2) and `TestLintCommentsCmd_OverARepository` / "a wrapped new comment fails with its file, line and text" and "a clean tree succeeds" cover it.
    - The git argv for an empty base is unchanged (`diff … HEAD` plus `ls-files --others`).

**Loosened checks:** none.
- The base-form step's old want of `added.go:3` for `Lint(dir, afterRename)` moved, with the same literal, to the `LintCommitRange` subtest.
- `Lint`'s want grew to cover the new working-tree behaviour.
- The loomcli subtest was renamed only, and its assertions are unchanged.

**Tautology check:** every changed or new want is a literal `file:line` key or an envelope string, never a value computed through the code under test.

**Findings:** none.

**Tool calls:**
- Read `lint_integration_test.go` lines 100–130 and `lintcomments_integration_test.go` lines 20–100. The diff pieces showed only the hunks, not the full assertion of the indexless step or the loomcli test's setup and `base` commit.
