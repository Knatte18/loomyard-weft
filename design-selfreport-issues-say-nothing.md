# loom anomaly issues do not say what went wrong

`selfreport-issues-say-nothing` · Note · bug, selfreport, loom

loom's anomaly self-report (`internal/loomengine/anomaly.go`, `internal/loomcli/selfreport.go`) files GitHub issues that do not say what went wrong.
Issues #366–#382 from 2026-10-05 show it.

- A recurring-finding issue (e.g. #370) carries only the ledger key (`d4-budget-test-off-by-one`), the Bouncer row and the round numbers.
  It omits the finding's text, the reviewers' reasoning per round, the path to the review files and the decision that closed the loop, so the issue cannot be acted on without the torn-down worktree.
- State, current producer and error print empty, and history prints "no history entries", even when the run had them.
  Empty fields should be dropped or filled, never rendered blank.
- Crash-resume issues (#376–#380) fired for the deliberate migration of four runs from the laptop to the Ryzen machine; a resume the operator or hub started on purpose (adopt, `lyx fabric reconcile`, `lyx loom start` after a takeover) is not an anomaly.
- Every issue gets the `bug` label, though a recurring finding or a budget exhaustion is a process signal, not a defect.

The issue should answer three questions on its own: what happened, why lyx considers it abnormal, and what someone should do about it (or that nothing is needed).
Recurring findings that ended CONVERGED, and runs that later landed, may be better as one summary per run than one issue per finding.
