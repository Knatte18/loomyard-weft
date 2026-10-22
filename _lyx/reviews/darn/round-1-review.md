---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loopwait.go:427-433,505-511
    summary: the dead-loop cause and trace_file call the pid record's last child in flight even when the status has moved on, so a loop that died between steps names a finished step and its trace
    fix: In deadLoopStop and loopExitStop, treat the recorded child as in flight only when the status file still reads the recorded producer and history length (the same comparison that sets obj.StatusMoved at loopwait.go:489, using readStatusFile(spec) before the failed stop is written). When the status has moved, leave "step X was in flight" and the trace id out of the detail and keep trace_file as spec.Loop.LogPath. Add a row to the subtest at loopwait_test.go:470-520 where the record names producer A, the status reads producer B with a moved history length, and assert the cause has no "was in flight" clause and the file is the loop log. The existing TraceKnown row sets the status producer to B and the record producer to A, so change its status to the record's producer and history length so it still pins the in-flight case.
  - id: F2
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shedverbs/loopwait.go:435
    summary: the changed detail clause "its step tree was still running and was killed" has no test, because the new cases use a fake pid with no process group and never reach killStepTree
    fix: Extend TestLoopIntegration_ChildTreeDiesWithLoop in internal/shedverbs/loop_integration_test.go to assert that the dead-loop stop's loop.detail (or the cause the hook receives) contains "its step tree was still running and was killed" after the loop is killed with a live child.
  - id: F3
    severity: LOW
    class: consistency
    origin: lens:code-tests
    location: internal/shedverbs/loopwait.go:455-459,502-511
    summary: the early-exit stop now hands the hook a loop-exited prefixed cause with the loop log as trace file, and no test asserts those arguments
    fix: Add a case to the existing early-exit test in internal/shedverbs/loopwait_test.go that sets spec.Hooks.AfterInterrupt and asserts the cause is the text loop-exited, a colon and a space, then loop ID exited before it recorded itself; its output is in LogPath, and that the file is spec.Loop.LogPath.
  - id: F4
    severity: NIT
    class: consistency
    origin: lens:code-tests
    location: internal/shedverbs/loop_integration_test.go:153-155
    summary: the integration assertion only checks the pid record's trace_id is non-empty, so a wrong id would pass
    fix: Compare record.TraceID with the trace id the in-flight child actually ran under (for example the id in the child's trace file name or the id the test's fake child reports) instead of only checking it is non-empty.
---

### [MEDIUM] Stale child is named as in flight

**Location:** internal/shedverbs/loopwait.go:427-433,505-511
**Issue:** The pid record keeps the last child's producer and trace id until the next child starts. When the loop dies between steps, the record names step A, the status has already moved to B, and the cause reads "step A was in flight under trace T" with `trace_file` set to A's trace, while the halt note's producer is B. The board entry asks for the trace of the step that was in flight; here that step had finished. `obj.StatusMoved` already computes the condition, but only after the detail is built. The new `TraceKnown` row pins the mismatch (record producer A, status producer B) as expected output. The `shedverbs/doc.go` sentence "producer of the in-flight child" has the same gap and needs the "when the status has not moved" qualifier in the same change.

### [LOW] Killed-tree wording is untested

**Location:** internal/shedverbs/loopwait.go:435
**Issue:** The rewritten clause is asserted by no test, before or after the change. The summary lists it as a behaviour of the stop.

### [LOW] Early-exit hook arguments are untested

**Location:** internal/shedverbs/loopwait.go:455-459,502-511
**Issue:** `loopExitedEarlyStop` goes through `loopExitStop`, so the hook's cause changed from the bare token to the prefixed detail. No test asserts the hook's arguments on this path.

### [NIT] Weak trace id assertion

**Location:** internal/shedverbs/loop_integration_test.go:153-155
**Issue:** `record.TraceID == ""` is the only check, so a trace id of the wrong value passes.

## Rejected

- Pattern-fit lens: `pattern/PATTERN-friction-capture.md` does not mention interrupted stops. The fork itself called it a gap in background text with the substance covered in `loomcli/doc.go` and `friction/doc.go`, not a violation of a listed constraint, and the stop already wrote a note before this change.
- Focus question on the panicking child reclassifying to `interrupted-stop`: not a defect. The summary and the decision record state it, and the board entry licenses one kind for stops without an envelope.
- Focus question on other `ClassifyHalt` callers and anomaly-kind consumers: none missed. `halt.go` is the only caller, `haltnote` treats the kind as an opaque string, and the reflection stencil lists no kinds.
