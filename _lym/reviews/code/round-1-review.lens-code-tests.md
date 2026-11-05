```yaml
lens: code-tests
outcome: reported
```

Lens: code-tests. I list each behaviour the diff adds or changes, then the test that pins it or none.

**Per behaviour**

| Behaviour | Pinning test |
|---|---|
| `spawnLoop` writes the starting mark and the exit mark with pid and status | `TestLoopIntegration_SpawnMarksTheLoopsExit` (tier 1, real `/bin/sh`, exit 3). The regex needs both marks, and the read happens after `<-exited`, so the mark-before-close ordering is pinned. The `exit status 0` branch and the failed start-mark write are not pinned (LOW, not filed). |
| `loopExitEvidence` reads only lines after the dead loop's last start mark | `TestUntilStop_DeadLoopStopsLoopExited`, row `LoopExitMarksAndStderr_OnlyTheDeadLoopsLines`. It holds an older loop's crash and `old-1` marks before the `dead-1` start. |
| `loopExitEvidence` drops other loops' marks that sit after the start mark | None. In the row, every `old-1` line comes before the `dead-1` start mark, so the `!HasPrefix(line, loopLogMarkPrefix)` exclusion never runs. |
| `loopExitEvidence` on a log that has content but no start mark for the loop (older binary) | None. The no-mark rows have an absent or empty log. The summary and `doc.go` both claim "an older loop's crash is never blamed on this one". |
| `loopExitEvidence` gives `loop exit: not recorded` | Pinned by the changed `wantCause` values in the existing rows and in the early-exit test. |
| `loopExitEvidence` excerpt after the start mark, with Go-crash handling | Same new row: `panic: boom`, then `goroutine 1 [running]:`, then `main.main()`. |
| `deadLoopStop` appends `step stderr:` from the recorded child's stderr | Same new row (`two | three | four`). Missing or unreadable stderr falls out as "" in the rows with no stderr file. |
| `recordedStopFiles` and `loop.stderr_path` in the envelope | Every row of the interrupt-hook table asserts `stderr_path`. The no-trace-id row expects "". |
| `loop.trace_copy` filled by `loopExitStop` through `copyTraceFiles` | None. No test in the diff asserts `trace_copy` on a dead-loop envelope. |
| `loopExitedEarlyStop` detail carries the evidence | `TestUntilStop_DeadLoopStopsLoopExited`, the early-exit case, expecting `; loop exit: not recorded`. Real marks on that path are not tested; it is the same function, so I don't file it. |
| `stepUntilStop` recovers a panic into a `loop-panicked` interrupted stop with an envelope | `TestLoop_InterruptedChildFillsTheErrorForm`, subtest "the loop panics while stepping". It asserts the exit code, kind, error text, hook cause, failed status, envelope file and removed pid file. |
| Panic recovery calls `killStepTree(*pid)` and writes the stack to the trace file | None. The fixture child never records a process group, so the kill is a no-op, and nothing reads the logged stack. |
| `stepUntilStop` refactor leaves the stepping logic unchanged | The existing `TestLoop_*` tests, unmodified. |
| Terminating signal: mark written, then `Raise` | `TestLoop_TerminatingSignalIsMarkedInTheLoopLog` (tier 0). It checks the exact log content and that the raised signal is SIGTERM. |
| Production `notifyTerminatingSignals` and `raiseSignal` | `TestLoopIntegration_ChildTreeDiesWithLoop`, changed to send a real SIGTERM. It asserts the next waiter's detail carries the signal mark and the killed step tree. |
| `recordTerminatingSignals` stops on return | Nothing asserts this. The unit fixture returns a nil channel for every other test, so only the no-signal path runs. Not filed. |
| `outputExcerpt` split out of `stderrExcerpt` | Existing `stderrExcerpt` tests, unchanged, plus the new row. |

**Findings**

1. Severity MEDIUM, class `scope`. Location: `internal/shedverbs/loopwait.go:553` (`loopExitEvidence`), test `internal/shedverbs/loopwait_test.go` row at diff lines 720-733.
   - Summary: the claimed no-start-mark behaviour has no test row. A log with content but no start mark for the loop must yield only `loop exit: not recorded` and no excerpt.
   - Fix: add a row to `TestUntilStop_DeadLoopStopsLoopExited` with `loopLog` set to another loop's crash lines and no `dead-1` start mark, expecting only `; loop exit: not recorded`.
   - Evidence: the `start >= 0` guard at `loopwait.go:566`. The only rows that set `loopLog` contain a `dead-1` start mark.

2. Severity MEDIUM, class `scope`. Location: `internal/shedverbs/loopwait.go:578-581` (`loopExitStop`).
   - Summary: `loop.trace_copy` is filled by new code, and the summary and `doc.go` both claim it. No test asserts it.
   - Fix: make a trace-listing row in the interrupt-hook table use real trace files under `t.TempDir()`. Assert that the envelope's `loop.trace_copy` equals the `StopFiles` copy path and holds their concatenation. Assert it is empty when the trace id or the listing is absent.
   - Evidence: the new `recordedStopFiles` and `copyTraceFiles` block at diff lines 652-656. The rows only assert `stderr_path`, and their trace paths such as `/logs/trace-7.log` don't exist on disk.

3. Severity LOW, class `scope`. Location: `internal/shedverbs/loop.go:196-199` (the recover in `stepUntilStop`).
   - Summary: the panic subtest doesn't pin the in-flight step tree kill or the stack in the trace file, both of which `doc.go` and the summary claim.
   - Fix: in the panic subtest, make the runner record a child with a process group before it panics. Assert the kill through the existing process-ended helpers, or drop the claim from `doc.go` and the summary.
   - Evidence: the panic runner at `loop_test.go` diff line 362 (`panic("boom")`) records no child, so `killStepTree` returns false at `loopenvelope.go:264`.

4. Severity LOW, class `scope`. Location: `internal/shedverbs/loopwait.go:569-573` (`loopExitEvidence`).
   - Summary: no row has another loop's mark inside the dead loop's span, so the `!HasPrefix(line, loopLogMarkPrefix)` exclusion is unpinned. In the new row, all `old-1` lines come before `dead-1`'s start mark.
   - Fix: add an `old-1` mark line between `dead-1`'s start and exit marks in the new row, and keep the expected cause unchanged.
   - Evidence: the row's `loopLog` at diff line 723-725.

On test economy, `TestLoopIntegration_SpawnMarksTheLoopsExit` covers the writing side and the dead-loop row covers the reading side, so I found no duplicate tests.

**Tool calls:** none. The manifest and the diff already held everything this lens needed.
