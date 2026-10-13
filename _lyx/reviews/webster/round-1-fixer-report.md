# Round 1 fixer-report

## Changes

- F1 (fixed with own change): `internal/shuttleengine/gate_pending_test.go` — `TestGatePending_MidTurnTicksNeverReevaluate_DeadlineUsesFinal` now sets `Forked` alongside `Gate` and `Final` on the entry and runs with `RunState.ForkSubagents` true.
  It asserts gate, forked and final calls are 0, 1 and 1, so swapping the `Forked` and `Final` assignments in `Run.evaluateGate` fails the test.
  The fix key offered this row or a `TestGateList_FinalReports` case; I extended the existing test, adding no new top-level test.

## Disputed

None.

## Deferred

None.
