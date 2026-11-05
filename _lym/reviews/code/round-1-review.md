---
verdict: APPROVED
findings:
  - id: F1
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loopwait.go:510
    summary: "The step stderr excerpt, loop.stderr_path and loop.trace_copy come from the recorded child even when the status file has moved past it, so a loop that dies between Drift-Await done and the Webster start reports the finished Drift-Await step's stderr and files as the cause"
    fix: "In deadLoopStop (loopwait.go:510-515) append the step stderr clause only while the recorded child is still in flight, the same inFlightAt condition that gates the in-flight clause just above it. In loopExitStop (loopwait.go:578-582) take stderrPath and traceCopy from recordedStopFiles only under the same condition, and leave obj.StderrPath and obj.TraceCopy empty otherwise. Add a row to TestUntilStop_DeadLoopStopsLoopExited where statusMoved is true, with a stderr file present, and assert no step stderr clause, an empty loop.stderr_path and an empty loop.trace_copy."
  - id: F2
    severity: MEDIUM
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loop.go:335
    summary: "Panic recovery covers only stepUntilStop, so a panic in OnStart, AfterInterrupt, OnExit, WriteFailedStop or the code after the stepping still ends the loop with no envelope, while doc.go says a loop ends without one only by a signal or a machine failure and the summary calls the path closed"
    fix: "Either move the recover to cover runLoop from after the pid file is written to the envelope write, turning a panic there into the same loop-panicked interrupted stop where the envelope can still be written, or narrow the claims. If the claims are narrowed, change the first added sentence of the doc.go paragraph (it begins A loop ends without an envelope only by) to name the panic outside the stepping and the early returns documented above it (lock not taken, teardown mark, undelivered envelope, dead loop pid file) as further ways, and change the runLoop doc comment to match."
  - id: F3
    severity: MEDIUM
    class: decision
    origin: handler
    location: internal/shedverbs/loop.go:427
    summary: "The board entry asks to make the path where a loop exits without an envelope write one; the diff instead decides that a terminating signal writes no envelope and only records a mark, so the entry's second ask is met by better reporting, not by an envelope, and the record carries no Added after Discussion entry for this choice"
    fix: "Leave the code in place and defer this finding to the run's parent: recording the choice that a signal ends the loop without an envelope, or refusing it, is done with lym loom decision add and lies outside the fixer's write surface."
  - id: F4
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shedverbs/loopwait.go:578
    summary: "loop.trace_copy is now filled on a dead-loop stop through copyTraceFiles, which doc.go and the change description claim, and no test asserts it"
    fix: "In TestUntilStop_DeadLoopStopsLoopExited, give one interrupt-hook row real trace files under t.TempDir() through w.traces, and assert that the envelope's loop.trace_copy equals the path StopFiles returns for the trace id and that the file holds the concatenated trace files. Assert loop.trace_copy is empty in the rows with no trace id or no listed file."
  - id: F5
    severity: MEDIUM
    class: scope
    origin: lens:code-tests
    location: internal/shedverbs/loopwait.go:566
    summary: "The claim that a log with no starting mark for the loop yields only the loop exit clause, so an older loop's crash is never blamed on this one, has no test row"
    fix: "Add a row to TestUntilStop_DeadLoopStopsLoopExited whose loopLog holds another loop's crash lines and marks and no starting mark for dead-1, and expect the cause to end with only the loop exit not recorded clause, with no loop log clause."
  - id: F6
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loop.go:351
    summary: "The panic recovery kills the step tree in pid.Child, which holds the previous finished step's record until the next child starts, so a panic before the new child is recorded can kill a recycled process group"
    fix: "In stepUntilStop, clear pid.Child to the zero record when a step's runChild returns, and write the cleared record to the pid file only if needed by the dead-loop stop; or call killStepTree in the recover only when the current iteration's callback has run, tracked with a local flag set inside that callback."
  - id: F7
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loop.go:427
    summary: "After a signal the goroutine writes the loop log mark before it calls raise while the main goroutine keeps running; the same sender signals the step child, so the loop can write its failed stop and run AfterInterrupt in that window, which the design says it avoids, and the mark then says the loop ends without an envelope"
    fix: "In recordTerminatingSignals set an atomic flag before the mark is written, and have runLoop skip the failed-stop write, the AfterInterrupt hook and the envelope write when the flag is set, so the process only waits for raise; alternatively make the main path block once the flag is set."
  - id: F8
    severity: LOW
    class: design
    origin: lens:code-correctness
    location: internal/shedverbs/loop.go:462
    summary: "When SIGHUP or SIGINT was inherited as ignored, signal.Reset puts it back to ignored, raiseSignal returns, the loop keeps running, and the loop log has already said it ends without an envelope"
    fix: "In raiseSignal, after sending the signal, wait briefly and exit the process with status 128 plus the signal number when it is still alive; or skip signal.Notify for a signal that was inherited as ignored, using signal.Ignored."
  - id: F9
    severity: LOW
    class: design
    origin: handler
    location: internal/shedverbs/loopwait.go:181
    summary: "A mark is appended to the loop log after the process died, so a partial last line of the loop's output with no newline glues to the mark, the line no longer starts with the mark prefix, and the exit status is lost from the evidence"
    fix: "Make appendLoopLog and the start mark write begin the mark with a newline when the loop log does not end in one, by reading its last byte before the append, so every mark opens its own line."
  - id: F10
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shedverbs/loop.go:351
    summary: "The panic subtest does not pin the in-flight step tree kill or the stack written to the trace file, both claimed by doc.go and the change description"
    fix: "In the panic subtest of TestLoop_InterruptedChildFillsTheErrorForm, make the runner record a child with a process group through the callback before it panics, and assert that group is killed with the existing process-ended helpers; or drop the kill and the stack claims from doc.go and the change description."
  - id: F11
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/shedverbs/loopwait.go:569
    summary: "No row has another loop's mark inside the dead loop's span, so the exclusion of other loops' marks after the starting mark is unpinned"
    fix: "In row LoopExitMarksAndStderr_OnlyTheDeadLoopsLines, add an old-1 mark line between dead-1's starting mark and its exit marks in loopLog, and keep the expected cause unchanged."
---

### [MEDIUM] Stderr excerpt and files come from a step that already finished

**Location:** internal/shedverbs/loopwait.go:510, internal/shedverbs/loopwait.go:578
**Issue:** The board entry's second ask is a loop that exits between Drift-Await done and the Webster start. There the pid file still records Drift-Await as the child, and the status file has moved past it, so the in-flight clause is left out. The new step stderr clause, loop.stderr_path and loop.trace_copy are filled from the recorded trace id without that check, so the stop names a finished step's stderr tail and files as the cause of the exit, in exactly the scenario the entry describes.

### [MEDIUM] Panic recovery covers only the stepping

**Location:** internal/shedverbs/loop.go:335
**Issue:** The recover sits in stepUntilStop. A panic in the start hook, the interrupt hook, the exit hook, the failed-stop write or the code that builds the envelope still ends the loop with no envelope. doc.go says a loop ends without an envelope only by a signal or a machine failure, and the change description calls the path closed, so both claim more than the code does.

### [MEDIUM] The entry's second ask is met by reporting, not by an envelope

**Location:** internal/shedverbs/loop.go:427
**Issue:** The entry asks that a loop which can exit between Drift-Await done and the Webster start without an envelope write one. The diff finds no such path in the loop's own code, closes the panic path, and decides that a terminating signal writes no envelope and leaves the stop to the next waiter. The reasoning is recorded in the change description, which is the writer's own claim. The decision record has no Added after Discussion entry for it, so the departure from the entry is unrecorded. Per the rubric it is deferred to the run's parent, and the code stays as it is.

### [MEDIUM] loop.trace_copy has no test

**Location:** internal/shedverbs/loopwait.go:578
**Issue:** The new block in loopExitStop fills loop.trace_copy. The interrupt-hook rows assert only loop.stderr_path, and their trace paths do not exist on disk.

### [MEDIUM] The no-starting-mark behaviour has no test

**Location:** internal/shedverbs/loopwait.go:566
**Issue:** Only rows whose loop log holds the dead loop's starting mark exercise the excerpt. The guard that yields the first clause alone has no row, although the change description and doc.go both state it.

### [LOW] Recovery can kill a stale process group

**Location:** internal/shedverbs/loop.go:351
**Issue:** pid.Child is overwritten only when the next child starts. A panic between two steps kills the group of the step that already finished, which may have been recycled.

### [LOW] The signal path can still write a failed stop

**Location:** internal/shedverbs/loop.go:427
**Issue:** The goroutine writes the mark before it raises the signal, and the main goroutine is not held. The step child receives the same signal from the sender, so the loop can reach its failed-stop write and the interrupt hook before the process ends, which is the cancel-time halt note the design avoids.

### [LOW] An ignored signal leaves the loop alive with a false mark

**Location:** internal/shedverbs/loop.go:462
**Issue:** signal.Reset restores an inherited ignore for SIGHUP and SIGINT. raiseSignal then returns, the loop runs on, and the loop log says it ends without an envelope.

### [LOW] A partial output line hides the exit mark

**Location:** internal/shedverbs/loopwait.go:181
**Issue:** The exit mark is appended after the process died. If the loop's last output line has no newline, the mark joins it, fails the prefix check in loopExitEvidence, and the evidence reads loop exit not recorded although the status was written.

### [LOW] The panic subtest does not pin the kill or the stack

**Location:** internal/shedverbs/loop.go:351
**Issue:** The fixture child never records a process group, so killStepTree returns at its zero-group guard, and nothing reads the stack from the trace file. Both behaviours are claimed in doc.go and the change description.

### [LOW] Other loops' marks inside the span are unpinned

**Location:** internal/shedverbs/loopwait.go:569
**Issue:** In the new row every old-1 line precedes the dead-1 starting mark, so the branch that drops other loops' marks after the start never runs.

## Rejected

- The loop exit status is recorded only by the spawning waiter, so a SIGKILL that also ends that waiter still reads loop exit not recorded (code-correctness F6): doc.go states this limit, the signal marks cover the catchable signals, and nothing in the entry asks for more.
- `raiseSignal` as a violation of PATTERN-run-process-owner: it signals the loop's own process to re-raise a signal it received, not another run process (code-affected-set).
- Missing test for the exit status 0 branch of the exit mark and for the failed start-mark write: the regex test pins the mark format and the branches are one-line, so a row for each would only duplicate coverage.
- The focus question on the unreadable log and missing exit paths: the failure paths return the "not recorded" clause and never mask the original cause, so there is no defect.
