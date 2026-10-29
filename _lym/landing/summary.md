# Restore the verify-gate test for a misplaced //lym:guard marker

`TestVerifyGate_RoundScenario` in `internal/loomshed/verifygate_integration_test.go` again covers a misplaced `//lym:guard` marker reaching the verify gate's findings.

## What changed

- The subtest `fails with the file and line of a misplaced marker` is back, without the skip that `test-skips-os-tags` removed it for.
  It commits `a/a_test.go` with `//lym:guard` above `var misplaced = 1` on line 5.
  The round form, told no merge base, and the darn form, told a not-scoped answer and the first recorded pass as merge base, must each fail.
  Each failure must name `a/a_test.go:5` and the way to place the marker ("directly above").
- The subtest sits where it was, before `fails on a wrapped new comment before any test runs`, whose first commit rewrites `a/a_test.go` to a clean file for the steps after it.

## Why it can run now

The told command is the `code-burler-fix` key's `lym gate test --wait --gate code-burler-fix`, a computed scope.
That scope now takes its base from the newest whole-green ancestor in the green record, so a narrowed run reaches the guard step and its refusal reaches the findings through the log tail.

## Design decisions

- Test-only: no production code or doc changes.
  `internal/loomshed/verifygate.go` already documents that a misplaced marker reaches the findings with its file and line, and `internal/gatecli/doc.go` already documents the green-record base (D2).
