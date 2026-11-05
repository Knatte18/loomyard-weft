```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F4, F5, F7, F8, F9, F10, F11), looking for a concrete case where a proposal is wrong or breaks something.

**Findings**

1. **LOW** — `internal/shedverbs/doc.go` ("so the loop writes no failed stop, runs no interrupt hook and writes no envelope").
   - The sentence states an absolute that F7's proposal doesn't deliver.
   - `runLoop` checks `endedBySignal()` only after `stepUntilStop` returns and just before the envelope write.
   - A signal arriving after the first check but before the hook, or during `OnExit`, still lets the failed-stop write and `AfterInterrupt` run. Only the envelope write is skipped.
   - The `runLoop` doc comment words it as "once it has seen the signal", which is accurate. The `doc.go` wording drops that condition.

2. **LOW** — `internal/shedverbs/loop.go`, `recordTerminatingSignals` (the `arrived`/`raised` pair).
   - `signalled()` blocks on `<-raised`, which closes only after `raise(sig)` returns.
   - In production that is fine: `raiseSignal` either dies by the signal or exits after the grace.
   - A custom `Raise` that blocks, or never returns, would hang `runLoop` at the check instead of returning 1. The existing test's `Raise` returns at once, so nothing exercises this.

Cases I tried that held:
- **F1:** `inFlightAt(after)` is false for `loopExitedEarlyStop`'s childless record, matching the old empty `TraceID` result.
- **F4:** the existing `deadLoopWorld` and hook rows still match `inFlightAt`, with producer A and history 0.
- **F11:** the new old-1 line sits after the dead-1 start mark and is dropped by the `!HasPrefix(loopLogMarkPrefix)` case.
- **F8:** on Windows `Signal` errors, so the exit is still status 1 with no sleep.
- **F9:**
  - After a partial last line, the leading newline keeps the mark's prefix intact.
  - The empty line the newline leaves is dropped by `outputExcerpt`'s `TrimSpace`.
  - The check-then-append is a time-of-check/time-of-use window, harmless for the spawn-side exit mark because the process is already dead.
- **F10:** the new test panics via `Now`, which `runChild` calls after `record(child.Record())`. `pid.Child` is therefore set to the real group when the recover runs.

No tool calls made; my context already held everything this lens needed.
