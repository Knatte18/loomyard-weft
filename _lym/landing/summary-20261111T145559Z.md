# Test Finalize's flaky sink wiring and each rerun guard and outcome

The flaky rerun in Finalize's landing gate landed with two test gaps.
No test showed that `NewFinalize` wires the gate's flaky sink to the Finalize it returns,
and several guards and outcomes of `rerunTestFailure` had no test rows.
This change adds tests only; production code is unchanged.

## What changes

- `internal/landingshed/finalize_gate_test.go`: `TestNewFinalize_WiresTheGateFlakyPassesIntoTheLandingTexts` builds a Finalize through `NewFinalize`.
  It swaps only the gate's I/O seams for the fake verify tree and drives the gate through a failure that passes on its rerun.
  It then checks that the landing commit message and the GitHub merge body both list the failure under `## Flaky test failures`.
  Earlier tests built Finalize through `assembleFinalize` or a struct literal, so they never covered `NewFinalize`'s wiring.
- `internal/landingshed/verifygate_test.go`: `TestVerifyGate_RerunsATestFailureOnce` gains these rows:
  - a passing verify runs no rerun;
  - a failure whose log cannot be read, a log with no failure report, and a package that failed outside any test are not rerun;
  - a rerun whose runner crashes halts on the rerun's log, and the gate reports the crash;
  - a rerun whose shell could not start fails the gate on the rerun's log;
  - a rerun that the verify cannot run at all makes `check` return that error, and reports no flaky pass.

## Design decisions

- The wiring test uses a zero `fabricengine.Fabric` and a fake merge shuttle so that `NewFinalize` builds with no git and no session.
  Construction does no I/O on either.
- One rerun branch has no row: the fallback to the whole call when `gateslot.FailedTestsCall` cannot quote the failed tests.
  That call takes `runtime.GOOS`, and it fails only on Windows for a name holding `%`, so no Linux test can reach it.

## To check by hand

- The two real-run checks from the board entry are left to the next landing that meets a flaky test.
  The first is a tier0 test that fails once and lands with the `## Flaky test failures` section.
  The second is a .NET gate run whose report shows a VSTest failure's test name and error message.
