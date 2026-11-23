---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
  - id: F4
    disposition: fixed
---

## Changes

- F1, F4 (`internal/commentlint/diff.go`, `lint_integration_test.go`): an untracked `.go` file the base holds now gets its own path as base side, checked with `git cat-file -e <base>:<path>` through `gitexec.RunGit`.
  A row in `TestLint_OverARepository` untracks a committed legacy file and asserts `Lint` against the earlier base reports nothing for it.
  The test header now names `LintCommitRange`.
- F2 (`internal/loomshed/verifygate_integration_test.go`): the first comment-lint step leaves an untracked wrapped `.go` file in the worktree during the gate and asserts the findings do not name it.
- F3 (`internal/loomcli/lintcomments_integration_test.go`): the subtest name and the test doc describe `--base` as the working tree against a base commit.

Commits: `commentlint` F1+F4 (shared test file), `loomshed` F2, `loomcli` F3, then one `commentlint` follow-up from the self-check.
`lym gate test` passes at tier 0 and tier 1 for the three packages.

Gate repair: the comment lint flagged two comments this round added.
The `changedGoFiles` doc now breaks its untracked-file note at the sentence boundary, one sentence per line.
The `TestLint_OverARepository` doc is written as two sentences, one per line, each under the limit.

## Self-check

- fix-proposal, LOW: the `cat-file` probe ran for every untracked path, not only `.go` ones — fixed: the loop skips non-`.go` paths before the probe.
- fix-consistency, below LOW: the `TestLint_OverARepository` doc omitted the new step — fixed: the doc names it.
- fix-code-correctness: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.

## Rulings

None.
