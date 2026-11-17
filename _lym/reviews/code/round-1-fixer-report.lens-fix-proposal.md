```yaml
lens: fix-proposal
outcome: reported
```

Task: challenge the reviewer's proposals the fixer carried out as proposed (F1, F2, F4, F5, F6, F7, F8, F9), looking for a case where the proposal is wrong or breaks something.

**MEDIUM — F2, `reset.go` Reset: adding every `DirtyTracked` path to `own` after a stop.**
- The proposal turns `refuseForeignPaths` into a no-op whenever `settleRecordedStrands` stopped a strand.
- Case: an operator or parent has an unrelated uncommitted edit to `docs/x.md`. They run `lym weaver reset --stop-strand` over a live attempt. The edit is now in `own`, and `ResetCode(startSHA, own)` discards it with no refusal.
- Before the change, that edit was refused as "tracked changes the run did not leave".
- The flag only says "stop the strand", not "discard everything dirty". A safer split would keep foreign paths refused. For example, take `DirtyTracked` before the stop, and add only paths the strand wrote afterwards to `own`.
- The new test `StopStrandCountsTheStoppedAttemptsEditsAsTheRunsOwn` pins the over-broad behavior, since it has no foreign path.

**LOW — F5, `run.go` `settleAttempt`: the new state-write test only covers a failed outcome.**
- `settleAttempt` returns early when `out.Outcome == outcomeDone` and skips `stopAttempt`. A done outcome whose state save then fails leaves the strand running.
- The review's premise is that the strand is stopped all the same. The new row uses a `FAILED` report and so never reaches this path.
- A done outcome with a failed save has no row, and the code doesn't stop the strand there.

**LOW — F8, `run.go` `nextAttemptRung`: the fault retry has no cap.**
- A fault retries on its own rung with no allowance, and no count of faults exists. A persistent fault is retried on the same rung on every re-step and never climbs or stops.
- In the `runBatch` path, `failureDigest` is built only for failed or lost priors. The retry after a fault therefore tells the next attempt nothing about why the prior one ended.

**LOW — F6, `run.go` `truncateCause`: the bound can be defeated.**
- `LastIndex` of `"; way forward: "` keeps everything after the last marker whole. A cause with a long tail after that marker is not truncated at all.
- For example, a plan-conflict error joining several findings that each carry a way forward would pass through unbounded.
- The `StrandStopError` case is fixed, but `stuckReasonBytes` no longer bounds causes in general.

No findings for F1, F4, F7 or F9. F1's stop rung for the climb case is unused by re-entry. F7's lock holds through `archiveAndClear`, which keeps `*.lock` files.

Further tool calls: none. My context already held every changed file.
