# Restore the verify-gate test for a misplaced //lym:guard marker

`TestVerifyGate_RoundScenario` in `internal/loomshed/verifygate_integration_test.go` covers the guard scan over a narrowed diff again.
The subtest `fails with the file and line of a misplaced marker` had been deleted because it always skipped: the gate key's call carried no `--base`, so a narrowed run never reached the guard scan.
The key's computed scope now reads its diff base from the green record, so the scan runs and the case can return without a skip.

The test commits `a/a_test.go` with `//lym:guard` above `var misplaced = 1` on line 5.
Both gate forms must then fail, with findings naming `a/a_test.go:5` and "directly above", the way to place the marker:

- the round form, run with no merge base reader;
- the darn form, run with the first recorded pass as merge base and a not-scoped tree scope.

## Design decisions

- The subtest sits after the darn-form read-error table and before `fails on a wrapped new comment before any test runs`.
  That next step rewrites `a/a_test.go` to a clean file, so later steps see no misplaced marker.
- The subtest keeps the file's `tier1` tag and skips nothing.
  Like the rest of the scenario, it runs in the hermetic scratch repo.
- It asserts on the text "directly above", taken from the way forward `internal/gatecli/cli.go` prints.
  A rewording of that refusal needs this assertion updated.
- If the computed scope falls back to whole for the step, the guard scan still runs over the module and the subtest holds.
  A scope that skipped the scan entirely would fail it, which is the regression the test guards.
- The change is test-only.
  No docs change: `internal/loomshed/verifygate.go` and `internal/gatecli/doc.go` already describe this behavior.

## To check by hand

Nothing outstanding beyond running `lym gate test ./internal/loomshed`.
