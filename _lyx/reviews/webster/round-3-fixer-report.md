# Round 3 fixer report

Verdict was `APPROVED` with three LOW findings; all three are fixed, one commit each.

## Fixed

- F1 (`e9f352fe9`): `t.Parallel()` first in the `t.Run` subtest of `TestFindNewWrappedBreaks` (`internal/commentlint/lint_test.go`), `TestCommandFor` (`internal/impactset/derive_test.go`) and `TestTreeSitterLinkFindings` (`cmd/lyx/treesitterlink_test.go`).
  The three packages pass under `-race`.
- F2 (`04e45f144`): a `TestFindGuardTests` row in `internal/impactset/guards_test.go` for a test file that does not parse, asserting the `impactset: parse dir/p/p_test.go` error.
- F3 (`55ed37d3f`): a `TestFindNewWrappedBreaks` row in `internal/commentlint/lint_test.go` for a wrapped paragraph beside an indented code paragraph, flagging the wrapped paragraph's first line.
  F2 and F3 are rows in existing table tests, so no test function was added and the budget rows are unchanged.

## Deferred

Nothing.
