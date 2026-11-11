# Finalize flaky rerun: test rows for the sink wiring and each rerun guard and outcome

`finalize-flaky-test-gaps` · Task · enhancement, landing, gate · running · Drift-Recheck

## Seen (board tidy, 2026-11-11)

`finalize-flaky-rerun` landed as `dea07060` with findings its review left open under "To check by hand":

- No test covers that `NewFinalize` wires the gate's flaky sink to the Finalize it returns (`internal/landingshed/finalize.go`, `NewFinalize` and `assembleFinalize`).
- The guards that skip a rerun and the rerun's outcomes lack test rows each (`internal/landingshed/verifygate.go`, `rerunTestFailure`).
- Nothing ran beyond the unit tests: no real landing has had a tier0 test fail once and land with the `## Flaky test failures` section,
  and no real .NET gate run has shown a VSTest failure's test name and error message in the report.

## Idea

Add the missing test rows for the sink wiring and for each rerun guard and outcome.
The two real-run checks are left to the next landing that meets a flaky test, in this hub or a .NET hub; the orch reads its landing commit then.
