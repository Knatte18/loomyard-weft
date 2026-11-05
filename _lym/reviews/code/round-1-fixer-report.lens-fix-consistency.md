```yaml
lens: fix-consistency
outcome: reported
```

Task: check the fix-everything change set for disagreement with its surroundings (fix-consistency lens).

Findings:

1. **LOW** — `OnExit` contract still says "at every stop".
   - Changed text: the new `if endedBySignal() { return 1 }` checks in `runLoop` (`loop.go`).
   - Summary: after a terminating signal the loop returns before `OnExit` and writes no envelope, but its twin docs still promise `OnExit` runs at every stop.
   - Evidence: `loop.go:86`, `loop.go:179` and `doc.go:123` ("OnExit runs once at every stop"). `OnStart`'s registration is then never undone by `OnExit`. A signal ends the process anyway, so it is only stale text. A short "except when a terminating signal ends the loop" would close it.

2. **LOW** — `LoopSpec.Raise` comment is stale.
   - Changed text: `raiseSignal` (now sleeps `raiseGrace`, then exits with 128+signal).
   - Summary: the `Raise` field says nil "restores sig's default action and sends it to the process again".
   - Evidence: `loop.go:102`. It omits the new exit-after-grace behaviour. The `raiseSignal` doc itself is correct.

3. **LOW** — the new early-return list in `doc.go` reads as closed but is incomplete.
   - Changed text: `doc.go:111-112` ("an early return that refuses to step: the loop lock not taken, the teardown's mark, an undelivered envelope or a dead loop's pid file").
   - Summary: other `return 1` paths with no envelope are not listed.
   - Evidence: the loop-directory `MkdirAll` failure (`loop.go:186-188`) and the pid file write failure (`loop.go:211-214`). The reviewer's F2 fix named only the four documented ones, so this matches the review. It is still not "only by".

Checked, no disagreement:
- `recordTerminatingSignals` has a single caller, `loop.go:183`.
- Every mark writer now goes through the newline-safe path: the start mark in `loopwait.go:153`, the exit mark in `loopwait.go:181`, and the signal mark in `loop.go:454`. `appendLoopLog` at `loop.go:562` writes child output, which should not get a prefix.
- `loopExitedEarlyStop` passes a record with no child, so `inFlightAt` is false there and its detail and files are unchanged.
- The `doc.go` clauses on stderr, `trace_copy` and the hook's trace file agree with `deadLoopStop` and `loopExitStop`.
- No other doc or stencil mentions the dropped "stack goes to the trace file" claim. The `shed-template-driver` stencils only name `loop.trace_copy` and `loop.stderr_path` as evidence, which is still true.

Tool calls: 1. A repo-wide grep for the changed claims and symbols (`loop-panicked`, `stack goes`, `loopLogMark`, `appendLoopLog`, `loop.trace_copy`, `OnExit runs once`) outside `_lym` and `.lym`. My context had only the `internal/shedverbs` files.
