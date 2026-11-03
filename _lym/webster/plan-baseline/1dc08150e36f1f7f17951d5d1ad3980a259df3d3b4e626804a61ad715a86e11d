# Card 1 — gitrepo-lease-and-retry

**Create:**
- `plan:internal/gitrepo#PushWithRetryReasons` -> `func PushWithRetryReasons(push func() error, ref string, delays []time.Duration, sleep func(time.Duration), extraReasons []string) error`

**Edit:**
- `internal/gitrepo#retryablePushRefusal` -> `func retryablePushRefusal(err error, ref string, extraReasons []string) (line string, retryable bool)`
- `internal/gitrepo#PushWithRetry`
- `internal/gitrepo#ErrLeaseRejected`
- `internal/gitrepo#Repo.DeleteRemoteBranchLeased`
- `internal/gitrepo#TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine`
- `internal/gitrepo#TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound`
- `internal/gitrepo_test#TestDeleteRemoteBranch`
- `internal/gitrepo/doc.go`

**Intent:** Two gitrepo changes the remote-order work rests on (decision record D4 step 4 and D6).

1. `retryablePushRefusal` gains `extraReasons`: a reason on the `! [remote rejected] <src> -> <dst> (<reason>)` line for the pushed ref that contains one of them is retryable, beside `(failed)` and `cannot lock ref`.
   `PushWithRetryReasons` is the bounded retry with those extra reasons; `PushWithRetry` keeps its signature and calls it with none, so every plain push (`PushRebaseFree`, `Push`'s first attempt, `PushCoalesced`) classifies exactly as today: no new `ErrPushRetriesExhausted` in place of `ErrPushRejected`, and no line withheld from `Push`'s `pull --rebase` recovery.
   Only the archive-tag push (card 6) calls `PushWithRetryReasons`, passing `missing necessary objects`: the 2026-11-02 rejection comes from receive-pack's connectivity check reading a store behind its own advertisement (decision record, Constraints), so a retry within the archive-tag push's existing delays can land it.
   Bound: the predicate still reads only the `remote rejected` line naming the pushed ref, and the retry count stays the caller's delays, so a remote that lost objects outright still fails after the last attempt.
   Update the doc comments of `retryablePushRefusal`, `PushWithRetry` and the new `PushWithRetryReasons`, the `pushretry.go` file header (its "repeats a push only when" sentence also names a caller's extra reasons), and the retryable-forms sentence in `internal/gitrepo/doc.go`, which gains that a caller may add reasons through `PushWithRetryReasons`.
2. `Repo.DeleteRemoteBranchLeased`, when the push's stderr carries `leaseRejectedTrigger` ("stale info"), returns `fmt.Errorf("gitrepo: git push --delete (leased at %s): %w", expectSHA, ErrLeaseRejected)`: the same text as today's wrapped error, now matching `ErrLeaseRejected` under `errors.Is`.
   Every other failure keeps today's wrapped git error and never matches the sentinel.
   Update `ErrLeaseRejected`'s doc comment to name both callers, and the `UpdateRemoteBranchLeased` lease bullet in `internal/gitrepo/doc.go` to name `DeleteRemoteBranchLeased` beside it.
   `deleteRemoteBranch` in `internal/fabricengine/destroy.go` returns the error with `%w` kept, so a caller there tells a moved branch from a failed delete with `errors.Is`; the texts of `DiscardTaskBranch`'s and `Add`'s step-11 errors that wrap it keep their `leased at <sha>` detail.

Tests:
- `TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine` gains rows whose stderr holds `! [remote rejected] refs/tags/archive/s/abc -> refs/tags/archive/s/abc (missing necessary objects)` for that ref: with `extraReasons` `["missing necessary objects"]` it reads retryable, with nil `extraReasons` it reads not retryable, and the same reason on another ref's line reads not retryable with the extra reason passed.
- `TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound` gains two rows over a fake push that fails once with that line and then succeeds: `PushWithRetryReasons` with the reason asserts two attempts and a nil error; `PushWithRetry` asserts one attempt and that line's error returned unchanged.
- `TestDeleteRemoteBranch` gains a row deleting under a lease on a SHA the bare remote's branch no longer sits at, asserting `errors.Is(err, gitrepo.ErrLeaseRejected)`, the text still naming `leased at <sha>`, and the branch still on the remote.

**ImpactSummary:** Adds one retry entry point with caller-given extra reasons and re-signs one private predicate whose only production caller is that retry; `PushWithRetry` and every plain push keep today's classification. One public method's lease failure now matches a sentinel with its text unchanged; its only caller, fabricengine's `deleteRemoteBranch`, passes the error through, so no caller's text changes.

**Commit:** `1: gitrepo retries caller-named refusals and names a lost delete lease`
