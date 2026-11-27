```yaml
lens: code-correctness
outcome: reported
```

I checked every branch the diff changes for correctness and found nothing wrong.

**Branches and the cases I ran:**

- **`mapOutcome`, fresh `Run` → `join`, chair outcome `timeout`:** correct.
  - `join` calls `stopSeat(chair)` when the step did not end early (`internal/seatengine/engine.go:475-476`).
  - If that stop fails, `stopErr` comes back as an error and `Call` goes to `errorExit`, so the reason is never written.
  - If it succeeds, "was stopped" is true.
- **Required advisor ended the step (`stepEnded`), chair timing out at the same moment:** correct.
  - The chair is stopped at `engine.go:467`.
  - If that stop fails, `end` stays zero and `stopErr` is non-nil, which is again the error path.
- **Chair's end arrives just after the advisor's end signal (the inner `case end = &lt;-chairDone`):** correct. `stepEnded` stays false, so the stop at `:475-476` runs.
- **`Resume` of a live table, chair timing out:** correct. It reaches the same `join` (`internal/seatengine/resume.go:161`), so the stop rule is the same.
- **Writer ended the step:** correct. `writerExit` returns before the chair switch, so this reason is never reached.
- **`Run` returns early with `ErrSeatNotStopped`:** correct. That is the error path, not this reason.
- **Context cancelled:** correct. `cancelErr` returns before the reason is built.
- **Chair has `NotStarted` set, or outcome `died`:** correct. Both return engine-level errors before the reason.
- **Timeout with zero times, no states and zero limit:** correct. `timeoutDetail` is unchanged and renders "unknown" and "none recorded", which the test row pins.
- **A resume after the halt starts a fresh session:** correct.
  - Wait records `timeout` (`internal/shuttleengine/wait.go:465`), which `isTerminalOutcome` counts as terminal (`internal/shuttleengine/attach.go:602`).
  - `Run.Stop` therefore leaves the record as it is; `errLeaveRunRecord` turns into nil (`rundir.go:245-250`), and Stop then removes the strand.
  - The next `Probe` finds no live chair, so `Call` takes the fresh `Run`.
  - In shuttle, `resumableAfterInterruption` returns false for `timeout` (`internal/shuttleengine/crashresume.go:26-42`). There is no provider resume of the stopped session, so "starts the row over in a fresh session" is true.
- **`timeoutWayForward` constant:** correct. Its doc comment's "a resume never finds it live" holds, because a failed stop returns an error instead of this reason.

**Findings:** none.

**Tool calls:**
1. I grepped `isTerminalOutcome`, `errLeaveRunRecord` and the `OutcomeTimeout` finalize in shuttleengine. My context had `Run.Stop` but not whether a recorded `timeout` counts as terminal, or whether `errLeaveRunRecord` makes Stop's record update succeed.
