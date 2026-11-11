---
round: 1
focus:
  - "internal/landingshed/finalize_gate_test.go: does the new test prove NewFinalize wires the gate's flaky sink to the returned Finalize, so removing the wiring fails it?"
  - "internal/landingshed/verifygate_test.go: does the diff add a row for each rerun guard and each rerun outcome in rerunTestFailure, with none missing or duplicating an existing row?"
  - "internal/landingshed/verifygate_test.go: does the changed expectation on a first-run runner crash (crashed flag, no rerun) match what rerunTestFailure actually does?"
  - "Scope: does the diff stay to test additions, with no production code change the Sketch does not license?"
---

Diff touches two test files only. The Sketch asks for test rows for the sink wiring and each rerun guard and outcome.
The two real-run checks are explicitly left out of scope.
