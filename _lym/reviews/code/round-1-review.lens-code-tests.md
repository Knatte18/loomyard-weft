```yaml
lens: code-tests
outcome: reported
```

Task: code-tests lens. For each behaviour the diff adds or changes, I looked for the test row that pins it.

**Per behaviour, the test that pins it**
1. `Lint` with a base other than HEAD reads the working tree, uncommitted edits and untracked files included. Pinned by `TestLint_OverARepository`, step "the working tree form against a base sees committed and uncommitted changes": it wants `added.go:3`, `renamed.go:7` and `untracked.go:3` as literals (`internal/commentlint/lint_integration_test.go:90-98`). Also pinned by `TestLintCommentsCmd_OverARepository`, step "lint-comments --base lints uncommitted edits too", which compares the full finding strings as literals (`internal/loomcli/lintcomments_integration_test.go:119-133`). The summary says this step fails on the merge base, and that holds: under the old code `--base` read HEAD, which lacks `edited.go`.
2. `Lint` with an empty base reads as HEAD, the same as before. Pinned by step "the working tree form against HEAD…", which wants `renamed.go:7` and `untracked.go:3` (`lint_integration_test.go:81-89`). Also covered by the existing loomcli steps that run lint-comments with no `--base` (`lintcomments_integration_test.go:61-70, 86-92, 104-108`).
3. `LintCommitRange` keeps the base-to-HEAD comparison and ignores the working tree. Pinned by step "the commit range form sees committed changes but not uncommitted ones". It runs with an uncommitted edit to `renamed.go` and an untracked `untracked.go` present, and wants exactly `added.go:3` (`lint_integration_test.go:68-80`).
4. Both forms detect renames, so a pure rename adds no finding. Pinned by step "a pure rename flags nothing", which loops over `Lint` and `LintCommitRange` (`lint_integration_test.go:56-66`). The working tree is clean at that point, so `Lint`'s rename detection runs on the `git diff &lt;base&gt;` path.
5. The verify gate calls `LintCommitRange` instead of `Lint` (`internal/loomshed/verifygate.go:146`). **None.** Every comment-lint step in the verify gate tests commits its wrapped file before it calls the gate (`internal/loomshed/verifygate_integration_test.go:234-238, 259-262, 395-398`). No test calls the gate with an uncommitted or untracked wrapped `.go` file, so reverting line 146 to `commentlint.Lint` passes every loomshed test.
6. The `--base` flag's help text (`internal/loomcli/lintcomments.go:70`, the mutation survivor) is static help prose. No test can reasonably assert it, so this is no finding.
7. Loosened checks: none. The old "base form sees committed changes" step became the `LintCommitRange` step and now runs with uncommitted noise present, which makes it stricter. The other assertions that existed at the review base are unchanged.
8. Tautology: none. Every expected value is a literal.

**Findings**
- **LOW, design**, `internal/loomshed/verifygate.go:146`: no test pins that the verify gate lints the committed range rather than the working tree. If the call reverted to `commentlint.Lint`, every loomshed test would still pass.
  - Evidence: `internal/loomshed/verifygate_integration_test.go:234` and `:395`. Every comment-lint case goes through `f.commitFiles` or commits before calling `gate()`, and no case leaves an uncommitted wrapped `.go` file in the tree.
  - Fix: add an assertion to an existing verify gate comment-lint step. Leave an uncommitted or untracked wrapped `.go` file in the worktree, and assert the gate's lint findings do not name it.

**Tool calls**
- grep of the loomshed test files for wrapped-comment, uncommitted and untracked cases: the manifest held no loomshed test, and item 5 needed one.
- grep of the loomshed tests for "wrapped" and comment lint, plus a read of `verifygate_rerun_integration_test.go:180-205`: to check whether any verify gate test runs the lint with a dirty tree.
