```yaml
lens: code-correctness
outcome: reported
```

I reviewed the code-correctness lens: each changed function, branch and error path, run on one concrete input.

## Cases run

- **`pushRecordsSide` with the default backoff (3 delays).** A commit made during the push leaves the worktree unpushed. Attempt 0 then reaches `recordsSideNotLanded`, which looks the branch up on origin, finds it there, and returns an error wrapping `errRecordsUnpushedPastPush`. The loop sleeps `delays[0]` and pushes again. Attempt 1 finds nothing unpushed and returns nil. **Correct.**
- **`pushRecordsSide` when every push leaves the worktree unpushed.** Attempts 0, 1 and 2 sleep `delays[0..2]`. Attempt 3 hits `attempt == len(delays)` and returns the wrapped error. That is 4 pushes in all, bounded, with no index past the slice. **Correct.**
- **`pushRecordsSide` with an empty delays slice.** Attempt 0 hits `0 == len(delays)` and returns before indexing. **Correct.**
- **`pushRecordsSide` when a retry fails or defers.** On the retry, `pushSide` returns the classified error (rejected, gone, deferral `*PushFailedError`, takeover), and the loop returns it unchanged. **Correct.**
- **`pushRecordsSide` with a deferral on attempt 1 or later.** `started` is reset each attempt, so a deferred record written by attempt n has `LastSeen &gt;= started` and is found. **Correct.**
- **`pushRecordsSide` when `HasUnpushed` fails to spawn.** It returns the error wrapped. A non-zero git exit, such as an unresolvable `@{u}`, folds to `true` and so enters the lookup and retry path. **Correct, with one caveat:** a missing upstream would burn the backoff before failing. In practice `setUpstream` sets the upstream on first push (`internal/gitrepo/pushdestination.go:63`).
- **`pushRecordsSide` with nil `opts.BackoffSleep`.** It falls back to `time.Sleep`, the same as `settleRejection` and `awaitOrigin`. **Correct.**
- **`recordsSideNotLanded` with a fresh deferred record.** It returns `*PushFailedError` before the origin lookup. **Correct.**
- **`recordsSideNotLanded` when the branch is present on origin.** It returns `%w` wrapping `errRecordsUnpushedPastPush`, and `errors.Is` holds through the outer `fmt.Errorf("...: %w")`. **Correct.**
- **`recordsSideNotLanded` when the branch is gone from origin.** It returns the ended message. The only way to reach this with a nil push is `checkOrigin` seeing `view.ended`; a gone branch whose entry is not ended returns `PushFailureGone` from `pushSide` (`internal/fabricengine/pushclassified.go:517-518`). The doc claim holds, and the ended subtests in `internal/fabricengine/pushclasses_integration_test.go:702-744` still get "ended on origin". **Correct.**
- **`recordsSideNotLanded` when the `ls-remote` lookup fails with a transport error.** It returns the lookup error, does not retry, and the run fails. **Finding 1.**
- **`recordsSideNotLanded` when a persistent tracking-ref write failure leaves the worktree reading as unpushed.** Origin already holds the commits. After 4 pushes it returns "the records branch holds commits its push did not carry". **Finding 2.**
- **`recordsSideNotLanded` when `CurrentBranch` or `readFailedPush` fails.** It returns the error. This behaviour predates the diff. **Correct.**

## Findings

1. **LOW, design.** `internal/fabricengine/pushanchored.go:332-335`.
   - **Problem:** a transient transport failure of the origin lookup in `recordsSideNotLanded` fails the run at once. It is not retried over the backoff.
   - **Why it matters:** the entry's Expected says "a transient classification does not fail the run at Seed-IDE". The pre-push lookup retries a transport failure over the backoff, and treats a non-transport lookup error as unchecked and goes on (`internal/fabricengine/pushclassified.go:540-558`).
   - **Fix:** treat a transport failure of this lookup as retryable, the same way as `errRecordsUnpushedPastPush`.
2. **LOW, design.** `internal/fabricengine/pushanchored.go:311` and `:337`.
   - **Problem:** when a push landed but its tracking-ref write keeps failing (logged only, at `internal/gitrepo/push.go:186-189`), every retry pushes "up to date" and still reads as unpushed. After the backoff, the run fails with "the records branch holds commits its push did not carry", although origin already holds them.
   - **Why it matters:** the error misreports the cause.
   - **Fix:** compare the worktree HEAD with origin's head of the branch, through `originBranchHead`, before calling the side unlanded.

Tool calls:
- Read `resolvePushDestination` in `internal/gitrepo/pushdestination.go` to check whether a first push sets the upstream, which decides whether the no-upstream case can loop. My context lacked that function.
