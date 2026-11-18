---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: design
    location: internal/shuttleengine/stepstop.go#stopStepRunsWhere
    summary: The orphan selection is judged on the scan's read of run.json and never re-checked before stopRecorded, so a run that a standalone step or another process attaches between the scan and the stop is stopped while its new waiter lives.
    fix: In stopStepRunsWhere, re-read each candidate's record with loadRunState immediately before calling stopRecorded, and skip the candidate (no stop, no listing) when the re-read record no longer reads running or selects no longer accepts it; this makes StopOrphanedStepRuns drop a run whose record now carries a live attaching pid. Add one row to TestRunner_StopOrphanedStepRuns_StopsOnlyRunsWhoseWaiterIsGone, not a new test, that rewrites a gone row's record with os.Getpid() after the scan, for example from a fakeReed Status hook, and asserts that run stays running with no RemoveStrand call. Drop the sentence in _lym/landing/summary.md claiming no live run can match at loop start, or narrow it to the loop's own steps.
    origin: handler
  - id: F2
    severity: LOW
    class: design
    location: internal/shuttleengine/stop_test.go:353
    summary: The empty-root no-probe assertion is vacuous, because fakeReed.Status keeps its last queued result, so StatusQueue stays at length 1 whether or not Status was called.
    fix: In TestRunner_StopOrphanedStepRuns_StopsOnlyRunsWhoseWaiterIsGone, replace the len(fake.StatusQueue) != 1 check after the empty-root call with a check that fake.CallLog holds no Status entry, keeping the empty-report and nil-error checks.
    origin: assist:assist
  - id: F3
    severity: LOW
    class: design
    location: internal/loomcli/looparm.go#orphanedStepsNote
    summary: No test pins the reconcile hook's error path; the looparm_test.go hooks table that runs the failing-stop row through watchdogKillNote and orphanedStepNote does not include orphanedStepsNote, and cli_test.go drives only a successful report.
    fix: In internal/loomcli/looparm_test.go, add orphanedStepsNote to the existing hooks table as a third row, adapting the table's stop func to the stepsDir signature, so the existing report rows and the failing-stop row run through it; add no new test function.
    origin: lens:code-tests
  - id: F4
    severity: LOW
    class: consistency
    location: internal/shedverbs/doc.go:203
    summary: The new paragraph says a stop whose hook failed or a step that ended in an error does not leave its runs to the re-step, but the package doc also uses re-step for the loop's immediate transient re-step, which runs with no reconcile.
    fix: In internal/shedverbs/doc.go, in the paragraph that opens Every loop calls Spec.Loop.ReconcileOrphanedSteps, replace the re-step with the next loop's first step.
    origin: lens:code-affected-set
  - id: F5
    severity: NIT
    class: consistency
    location: internal/loomcli/looparm.go#stepStopNote
    summary: The shared note renderer words a stop error as could not stop the step's runs, and the reconcile note, which covers every gone step, now logs reconciled the steps whose process is gone followed by could not stop the step's runs.
    fix: In stepStopNote's error branch, change the wording to name no single step, for example could not stop the runs followed by the error, and update the matching want string in looparm_test.go's failing-stop row.
    origin: lens:code-correctness
  - id: F6
    severity: NIT
    class: consistency
    location: internal/shuttleengine/stepstop.go#StopOrphanedStepRuns
    summary: The doc line saying a run qualifies when its waiting process is gone or the record names none reads as a record naming no step, which the doc's later sentence says is left alone.
    fix: In StopOrphanedStepRuns' doc comment, change or the record names none to or the record names no waiting process.
    origin: handler
  - id: F7
    severity: NIT
    class: consistency
    location: internal/shuttleengine/stop_test.go:358
    summary: The row named gone seeds PID -1, which fails runIsLive's PID > 0 check like the unrecorded row's PID 0, so no row exercises a recorded positive pid whose process is dead.
    fix: Give the gone row a positive pid that is not alive, using the isAlive stub activity_test.go uses, or the pid of a child process the test starts and reaps, and keep the row's expected outcome stopped.
    origin: lens:code-tests
---

### [LOW] A run attached between the scan and the stop is still stopped

**Location:** internal/shuttleengine/stepstop.go#stopStepRunsWhere

**Issue:** `StopOrphanedStepRuns` selects on `!runIsLive(rs)` from the scan's read.
The stop happens later, after a reed probe and `sessionMayBeHeld`.
`stopRecorded` (`stop.go:87-101`) re-reads the record but checks only that the outcome is not terminal.
`Attach` rewrites the record's pid to the attaching process (`attach.go:333-338`).
The loop lock excludes the loop's own children, but not a standalone `lym loom step`, which takes only the run lock (`step.go` non-loop branch).
It does not exclude another process attaching either.
So a run attached inside that window loses its live agent.
The window is short and needs a concurrent attacher, hence LOW.
The change description's claim that no live run can match at loop start holds only for the loop's own steps.
The assistant raised the same race at MEDIUM with a run-lock fix.
A re-check on the re-read record closes it without widening the loop's locking, and the short window keeps it LOW.

### [LOW] The no-probe assertion cannot fail

**Location:** internal/shuttleengine/stop_test.go:353

**Issue:** `fakeReed.Status` (`fakes_test.go:150-165`) pops the queue only while it holds more than one result.
The fixture queues exactly one, so `len(fake.StatusQueue) != 1` stays false whether or not reed was probed.
The fake records every probe in `CallLog`.

### [LOW] The reconcile hook's error rendering has no test row

**Location:** internal/loomcli/looparm.go#orphanedStepsNote

**Issue:** `orphanedStepsNote` passes the stop's report and error to `stepStopNote`.
If it dropped the error, no test would fail.
`looparm_test.go`'s hooks table runs the failing-stop row through `watchdogKillNote` and `orphanedStepNote` only.
`cli_test.go`'s wiring subtest returns a nil error.

### [LOW] Re-step in the new doc paragraph is ambiguous

**Location:** internal/shedverbs/doc.go:203

**Issue:** The package doc already uses re-step for the loop's one immediate re-step after a transient error.
That re-step runs inside `stepUntilStop`, after the reconcile at `loop.go:243`.
A transient-error step's runs therefore do reach it, which contradicts the new sentence read that way.

### [NIT] The stop-error wording names one step for a multi-step reconcile

**Location:** internal/loomcli/looparm.go#stepStopNote

**Issue:** `orphanedStepsNote` reuses `stepStopNote`, whose error branch says the step's runs.
The reconcile's loop-log mark then reads as a contradiction of scope.
The code-affected-set lens raised the same point.

### [NIT] Names none in the StopOrphanedStepRuns doc is ambiguous

**Location:** internal/shuttleengine/stepstop.go#StopOrphanedStepRuns

**Issue:** The sentence's subject is a record naming a step, so names none can read as naming no step.
The doc then says a run naming no step is left alone.
`shuttleengine/doc.go` words the same rule clearly as the record names none of a waiting process.

### [NIT] The gone row does not test a gone pid

**Location:** internal/shuttleengine/stop_test.go:358

**Issue:** `runIsLive` is `PID > 0 && isAlive(PID)` (`activity.go:169-171`).
PID -1 short-circuits before `isAlive`, the same branch the unrecorded row takes.
The shared predicate is covered elsewhere, so this is only a row that does not test what its name says.

## Rejected

- Holistic concern that every loop start now stops live sessions a later step would reattach to, such as the PR-Review and sampler sessions their docs leave running on a cancelled context: rejected. Each loop path that leaves such a run without an envelope already hands its trace id to `OnStepOrphaned`, which stops it. That covers a child exiting, a panic and a dead loop. The board entry asks for the error-stop leftover to be stopped too.
- `TestSpecFor_ScratchAndFrictionDir` needing a `//testtiming:keep` marker: rejected, since it already carries one (`cli_test.go:286`).
- The new loop, loopwait and stop tests as redundant: rejected. The redundancy report lists each as unique, and their assertions pin behavior no covering test checks.
- Code-scope lens: no findings, and none rejected.

Focus questions settled:
- The orphan check reads only run records and waiter liveness, with no status error text. A record mid-write is read under its lock. A waiter alive at the scan is left alone; F1 covers a waiter that appears after the scan.
- The error-text recognition is gone with no caller left, and every loop past `loopStartRefusal` reconciles before its first child, a loop after an ordinary stop included.
- The loomcli wiring reports runner-build and stop errors in the loop-log note rather than swallowing them.
