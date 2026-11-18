---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    location: internal/shedverbs/loop.go:243
    summary: Startup reconciliation can stop a shuttle run that a concurrent standalone step has reattached.
    fix: In runLoop, acquire spec.LockPath before invoking ReconcileOrphanedSteps and hold it across reconciliation. If another step holds it, report busy without reconciling or launching a child; handle acquisition errors explicitly. Release the run lock before launching the first child. Extend the loop reconciliation test with a held-run-lock case that asserts the hook and child are not called.
    origin: lens:code-correctness
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/stop_test.go:353
    summary: The empty-root assertion cannot detect an unwanted reed probe.
    fix: In TestRunner_StopOrphanedStepRuns_StopsOnlyRunsWhoseWaiterIsGone, replace the StatusQueue length check after the empty-root call with an assertion that fake.CallLog contains no Status entry. Keep the existing empty-report and nil-error assertions; no new test is needed.
    origin: lens:code-tests
---

### [MEDIUM] Reconciliation does not exclude a standalone step

**Location:** internal/shedverbs/loop.go:243

**Issue:** The loop lock does not exclude a standalone step holding the separate shed run lock.
`Shed.Step` acquires that run lock in `internal/shedengine/run.go:595`.
The pre-step probe in `internal/loomcli/arm.go:606` also checks only the run lock.
Reconciliation can select an orphan in `internal/shuttleengine/stepstop.go:116`,
then a standalone step can attach and persist its live PID in `internal/shuttleengine/attach.go:333`.
Reconciliation subsequently calls `stopRecorded` at `stepstop.go:143`.
That function rereads the record but does not recheck the waiter before stopping and removing the strand.
The standalone step loses its live agent even if the loop's first child later refuses as busy.
Holding the loop lock alone therefore does not establish the new hook's no-step-in-flight precondition.

### [LOW] The no-probe test observes an unchanged queue

**Location:** internal/shuttleengine/stop_test.go:353

**Issue:** The fixture starts with one status response and asserts that one remains after the empty-root call.
`fakeReed.Status` retains its last response in `internal/shuttleengine/fakes_test.go:161`.
An unwanted status probe therefore leaves the assertion green.
The fake records every probe in `CallLog`, which can distinguish the required behavior.

## Review coverage

The committed range is `3746960e13964897b204f08b3ad79ca499e27cb5..HEAD`.
All required lenses returned and their reports are saved beside this report.
The board goal, decision record, focus directive, diff pieces, and commit messages were reviewed.
No scope or affected-set findings were retained.

The selector reads run records and waiter liveness without interpreting status error text.
Locked atomic record reads prevent observing a partial write; F1 concerns ownership changing after a complete read.
Every admitted loop invokes the hook before its first child, including starts after ordinary stops.
Nil locations leave the hook unwired, and scratch and friction directory wiring remains intact.
Stop errors are rendered into the loop note rather than silently discarded.
The package documentation matches the intended flow, subject to F1's concurrency defect.
`TestSpecFor_ScratchAndFrictionDir` already carries its keep marker; no additional marker is needed.

## Validation

Attempted `lym gate test --wait --tier 0 ./internal/shuttleengine ./internal/shedverbs ./internal/loomcli`.
The gate failed before running tests because its holder record under the hub's `_bolt/.lym/gate` is read-only here.
This is an execution limitation, not a finding against the diff.
