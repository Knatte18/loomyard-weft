Task: check the correctness of the changed code (F1, F3, F4, F5) by running concrete inputs through it, and check that each added test fails with its defect in place.

**Findings**

**LOW** — `internal/driftengine/expect.go`, `case rec.Mismatch` in `pass`
- The halt is placed ahead of the `ScopeOff` and `Deferred` cases.
- `doneUnsealed` writes only when `!rec.Scope.Unsealed`. A run that already holds `Unsealed`, with a hand-edited record, needs no write, but it now halts for the parent where it used to be Done.
- This is a narrow, behaviour-only difference. The `ScopeOff` and `Deferred` cases first record `Unsealed`, so an already-Unsealed run reaches them only after a resume.
- No test pins it.

No other findings.

Traces (summary of the fork's report):
- F1: with `clearable` reduced to `Skip == "" && BaselineFault == ""`, a records or seal fault plus an unexpected cell fails the gate with the findings; both new rows fail with the defect in place.
- F3: for the line `=1` and pattern `^(?P<key>\w*)=(?P<value>.*)$` the line is unkeyed with `Line: 1`; the new row fails with the defect in place.
- F4: `case rec.Mismatch` returns a Stuck with the decision prefix only; the `TheRecordsDifferFromTheirCommit` row fails without the branch; `Amend --restore` at a blocked Drift-Expect passes `admitAmend`.
- F5: `runAgent` keeps `openedAt` and closes the mark as `SessionMark{OpenedAt: openedAt}`; `pass` seals without a spawn only when `!Open && OpenedAt != ""`; the new row and the `AgentWritesAndTheFileIsSealed` assertion fail with the defect in place.

Tool calls: none made. Everything the lens needed was already in context.
