# Fixer-report, round 1

## Changes

- F1 (fixed as proposed, commit `7c9278b1f`): `internal/gatecli/cli_test.go` gains a subtest inside `TestGateTest_RefusesBeforeResolvingTheHub`, with no new top-level test.
  It runs `RunCLI` with `test --help` and asserts the output holds `for up to ` followed by `shuttleengine.DefaultAwaitCap.String()`, which pins the cap `Command` hands `newCommand`.
  `lyx gate test ./internal/gatecli` is green.
- Self-check follow-up (commit `47f9bbaa9`): the file header of `cli_test.go` now also names the slot wait its help text pins.

## Disputed

None.

## Deferred

None.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: LOW, the `cli_test.go` header and test name no longer describe the help subtest; fixed by widening the header (fix as proposed). The test name is left, since the review allows no new top-level test and a rename is outside the finding.
- fix-proposal: LOW, the same header and test-name mismatch; fixed with the same header edit.
- fix-affected-set: no findings.
