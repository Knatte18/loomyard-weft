# landingshed: test the flaky sink wiring and each rerun guard and outcome

Adds the test rows that the finalize flaky-rerun change left uncovered.
Only tests change; no production code is touched.

## Changes

- `internal/landingshed/finalize_gate_test.go`: `TestNewFinalize_WiresTheGateFlakyPassesIntoTheLandingTexts` builds a Finalize through `NewFinalize`.
  A gate failure that passes on the rerun must then show in both the landing body and the landing commit message, under the flaky section.
  This pins that `NewFinalize` connects the gate's flaky sink to the Finalize it returns.
- `internal/landingshed/verifygate_test.go`: `TestVerifyGate_RerunsATestFailureOnce` gains rows for the guards that skip a rerun:
  - a passing verify;
  - a failure whose log cannot be read;
  - a log with no failure report;
  - a package that failed outside any test.
- The same test gains rows for the outcomes of a rerun:
  - a runner crash;
  - a shell that could not start;
  - an error from the verify itself, which must surface as the gate's error.
- The test now also checks the `crashed` value `check` returns, including for a first-run runner crash.

## To check by hand

Nothing beyond the unit tests has run.
Two real-run checks remain for the next landing that meets a flaky test, in this hub or a .NET hub:

- A real landing where a tier0 test fails once and lands with the `## Flaky test failures` section.
- A real .NET gate run showing a VSTest failure's test name and error message in the report.
