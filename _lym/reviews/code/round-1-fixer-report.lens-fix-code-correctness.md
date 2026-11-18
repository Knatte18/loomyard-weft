```yaml
lens: fix-code-correctness
outcome: reported
```

Task: check the correctness of the F1–F3 change set (`pushanchored.go`, `doc.go`, `fabric.go`, and the integration test) through the fix-code-correctness lens.

no findings

Traces I ran:
- **Spent backoff:** the hook commits on every push, so there are four pushes and three sleeps. The attempt-3 check, `attempt == len(delays)`, calls `recordOwedRecordsPush` with `PushFailureDeferred`. The error comes back as `fmt.Errorf("…: %w", *PushFailedError)`. `pushFailuresOf` unwraps it through `errors.Unwrap`, so `IsPushOwed` reads true.
- **Retry earlier in the loop:** `errRecordsUnpushedPastPush` is returned unrecorded before the last attempt. A deferred record written at the end cannot match a later push's `started` check, because its `LastSeen` is older.
- **Transport lookup failure:** `branchGoneOnOrigin` fails on `ls-remote` with a `*GitError` carrying "connection refused". `IsTransportFailure` is evaluated on the unwrapped `err`, and `recordOwedRecordsPush` returns a *PushFailedError of class transport. `errors.Is(notLanded, errRecordsUnpushedPastPush)` is false, so the loop returns it directly with no sleep.
- **Callers:** `recordsSideNotLanded` has one caller, which was updated. `opts.RunSlug` is assigned before the closure runs, so the record's `Runs` is correct. For the prime it is empty, which `IsPushOwed` does not read.
- **Tests fail without the fix:** with the defect in place, both new rows fail their `errors.As(*PushFailedError)` assertion, because the error was a plain one. The `sleeps == 1` assertion pins the re-push's use of `opts.BackoffSleep`. The gate log shows all three subtests passing.

Minor, not a finding: a failed `CurrentBranch` in `recordOwedRecordsPush` returns `cause` unrecorded. This cannot occur on either path, since `recordsSideNotLanded` has just read the branch.

Tool calls made: none; my context already held every changed file and the gate result.
