```yaml
lens: code-correctness
outcome: reported
```

**Lens: code-correctness. Per-function list (case run, result)**

- `stepUntilStop`, panic with a child in flight: the `pid.Child` set by the callback is killed, the stop is interrupted, cause is `loop-panicked: boom`, and `runLoop` writes the failed status and the envelope. Correct.
- `stepUntilStop`, panic before this step's callback ran (inside `runChild`, or in `readStatus`): `*pid` still holds the previous, finished step's child record, and `killStepTree` kills that stale group. Wrong, see F2.
- `stepUntilStop`, panic with a nil or empty value: `recovered != nil` holds for Go 1.21 and later (`PanicNilError`). Correct.
- `stepUntilStop`, panic outside the function (`OnStart`, `AfterInterrupt`, `OnExit`, `WriteFailedStop`, `interruptedRecord`, any code after the stepping): no recovery. Wrong, see F1.
- `stepUntilStop` normal paths (continue, halted, condition, busy, transient re-step, error, status-missing, status-unreadable re-step): same logic as the removed inline loop. `restepped`, `lastKnown` and `before` move into `run`, and `traceID` is hoisted so the defer can read it. Correct.
- `recordTerminatingSignals`, one SIGTERM while stepping: the mark is written and `raise` restores the default action and re-sends. Mostly correct, but the main goroutine keeps running while the mark is written, see F3.
- `recordTerminatingSignals`, signal arrives after `stopNotify`: the default action kills the process, no mark. Acceptable.
- `recordTerminatingSignals`, signal races with normal completion: both channels can be ready, so the mark "ends without an envelope" may be written for a loop that then writes its envelope. Cosmetic, under F3.
- `raiseSignal`, SIGHUP or SIGINT inherited as ignored (waiter under `nohup`): `Notify` enables the signal, and `signal.Reset` returns it to ignored. `self.Signal` succeeds, the process lives on, and the log holds a false "ends without an envelope" mark. Wrong, see F4.
- `raiseSignal`, platform that cannot signal itself (Windows): falls to `os.Exit(1)`. Correct.
- `spawnLoop`, start mark write fails: logged as a Warn, spawn continues, evidence then reads "not recorded". Correct.
- `spawnLoop`, `cmd.Start` fails: a start mark with no exit mark stays. Harmless, no pid file exists.
- `spawnLoop`, breakaway retry: one start mark, because the retry is inside `start`. Correct.
- `spawnLoop` exit goroutine: the mark is written before `close(exited)`. `takeSnapshot` reads `spawnExited` first, and `judgeSnapshot` (`loopwait.go:349`) holds a free lock until the spawn has exited. So a waiter that spawned the loop always sees the exit mark. Correct.
- `loopExitEvidence`, missing or unreadable log: `ReadFile` error ignored, so "loop exit: not recorded" with no excerpt. The original cause is not masked. Correct.
- `loopExitEvidence`, log with no start mark for this id: clause one only. Correct.
- `loopExitEvidence`, an older loop's lines before the start mark: excluded. Other loops' marks inside the range are dropped by the `loopLogMarkPrefix` check. Correct.
- `loopExitEvidence`, partial last line (a kill mid-write, no trailing newline): the next mark is glued to that line, so it is read as loop output. The exit mark is lost and "not recorded" is reported. Minor, not filed.
- `outputExcerpt`, empty or all-blank output: returns `""` (`lines[0:]` on a nil slice is safe). Correct.
- `deadLoopStop`, teardown mark: skips the new evidence. Correct.
- `deadLoopStop`, status already moved past the recorded child: still appends `step stderr:` and still sets `loop.stderr_path` and `loop.trace_copy`. Wrong, see F5.
- `loopExitedEarlyStop`: the `loopExitedEarlyStop` rec has an empty `TraceID`, so `recordedStopFiles` returns empty and no stderr path is set. Correct.
- `loopExitStop`: `copyTraceFiles` is guarded against an empty destination, and the files are listed through `traceFilesOf`. Correct.

**Findings**

**F1 — MEDIUM, design — `internal/shedverbs/loop.go:181-221`, `doc.go` added paragraph**
Only the stepping is covered by recovery. After the pid file is written, a panic in `OnStart`, in `AfterInterrupt`, in `OnExit`, or in `WriteFailedStop`, `interruptedRecord` or `stopDetail` still ends the loop with no envelope. A panic in a goroutine of `runChild` also cannot be recovered by the `defer` in `stepUntilStop`. The new doc says a loop ends without an envelope "only by a signal … or a failure of the machine", and `summary.md` says the path is closed. Both overclaim, and the board's "make that path write one" is only partly met. Evidence: `loop.go:216-218`, `223-321`.

**F2 — LOW, design — `loop.go:197`**
`killStepTree(*pid)` in the recovery kills whatever `pid.Child` holds. `pid.Child` is only reset by the next callback, so after a panic before the new child is recorded it holds the previous step's finished group, and the kill can hit a recycled process group. Fix: clear `pid.Child` when a step's `runChild` returns, or kill only when this iteration's callback has run.

**F3 — LOW, design — `loop.go:284-292`**
After a signal the goroutine writes the mark (mkdir, open, write) before it calls `raise`, while the main goroutine runs on. The same `runproc.End` signals the step child, so `runChild` can return interrupted and `runLoop` can write the failed stop, run `AfterInterrupt` (friction notes) and the envelope in that window. That is the cancel-time halt note the design says it avoids. The mark can also read "ends without an envelope" for a loop that did write one. Fix: set an atomic "signalled" flag first and make `runLoop`'s post-step writes wait on it, or block the main path after the flag is set.

**F4 — LOW, design — `loop.go:306-314`, `297-303`**
If SIGHUP or SIGINT was inherited as ignored, `signal.Reset` returns the signal to ignored. `raiseSignal` returns, the loop keeps running, and the log claims it "ends without an envelope". Fix: `raiseSignal` should end the process explicitly (`os.Exit(128+n)`) when it is still alive after a short wait, or `Notify` should be skipped for signals that were inherited as ignored.

**F5 — MEDIUM, design — `loopwait.go:510-515`, `578-582`**
The `step stderr:` excerpt, `loop.stderr_path` and `loop.trace_copy` come from `rec.TraceID` regardless of `rec.inFlightAt(status)`. The board's second ask is a loop that dies between Drift-Await done and the Webster start. In that case the recorded child is Drift-Await, which finished fine, so the stop's cause shows a finished step's stderr tail as "step stderr:" and points the operator at the wrong files. Fix: add the excerpt, stderr path and trace copy only while the child is still in flight, as the in-flight clause already does (`inFlightAt(before)`).

**F6 — LOW, scope (note for the handler) — `loopwait.go:174-183`, `doc.go`**
The exit status is recorded only by the waiter that spawned the loop. When the same cancel or teardown kills the waiter, as with `runproc.End` on the whole run, only a signal the loop caught (SIGTERM, SIGINT, SIGHUP) is recorded. A SIGKILL or a machine failure still reads "loop exit: not recorded", so the operator still cannot tell a crash from an external kill in that case. The doc states this honestly, so it is a limit, not a defect.

**Tool calls**
1. `grep` on `loopwait.go` for `spawned`, `waitDead` and related names: to find where the waiter waits for the spawn's exit, which my context lacked.
2. `Read` of `loopwait.go:240-365`: to confirm `takeSnapshot` and `judgeSnapshot` hold the free-lock decision until the spawn has exited.
3. `Read` of `loopwait.go:496-626`: to check the teardown branch and the in-flight conditions around the new evidence calls.
