# Fixer report — plan, round 1

## Changes

- F1 (fixed as proposed): in `_lyx/plan/00-overview.md`'s `## verify:` block, the untagged test line is now `go test ./internal/boardcli/... ./contracts/... ./cmd/lyx`.
  Every other line and the order are unchanged.
  The premise holds: `TestNoDeniedRecovery_ScannedSurfaceHasNone` (`cmd/lyx/norecoverydeny_test.go`) and `TestCLIForm_EveryCommand` (`cmd/lyx/clitree_test.go`) carry no build tag, so they run only in the untagged tier of `./cmd/lyx`.

## Disputed

None.

## Deferred

None.
