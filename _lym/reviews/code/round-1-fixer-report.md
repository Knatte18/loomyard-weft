# Fixer-report, round 1

## Changes

Commit `a9768a3c` fixes both findings; the F2 rows sit in the same test table as the F1 rows, so the two share one commit.

- F1, fixed as proposed: `gitBranchReadOptions["--verbose"]` and `gitBranchReadShortOptions['v']` are now `false` in `internal/burlerengine/cluster.go`, so `-v` passes only with no operand or beside a true list-mode option.
  `internal/burlerengine/doc.go` names `-v` beside `-a`, `-r` and `--format`.
  `TestFirstMutatingGitCommand` gains the mutating rows `git branch -v foo`, `--verbose foo` and `-vv foo`, and the non-mutating rows `git branch -v` and `-v -l feat*`.
- F2, fixed as proposed: `TestFirstMutatingGitCommand` gains `git branch --format -x`, `--points-at -x` and `--` as not mutating, and `git branch foo --` as mutating.

`lym gate test ./internal/burlerengine` passes.

## Self-check

No fork finding.

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

The lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

None.

## Deferred

None.
