# Card 1 — gitrepo-lease-and-retry

**Edit:**
- `internal/gitrepo#retryablePushRefusal`
- `internal/gitrepo#Repo.DeleteRemoteBranchLeased`
- `internal/gitrepo#Repo.PushRebaseFree`
- `internal/gitrepo#TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine`
- `internal/gitrepo#TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound`
- `internal/gitrepo_test#TestDeleteRemoteBranch`
- `internal/gitrepo/doc.go`

**Intent:** Two gitrepo changes the remote-order work rests on (decision record D4 step 4 and D6).

1. `retryablePushRefusal` also reads a `! [remote rejected] <src> -> <dst> (missing necessary objects)` line for the pushed ref as retryable, beside `(failed)` and `cannot lock ref`.
   The 2026-11-02 rejection comes from receive-pack's connectivity check reading a store behind its own advertisement (decision record, Constraints), so a retry within the archive-tag push's existing delays can land it.
   Bound: the predicate still reads only the `remote rejected` line naming the pushed ref, and the retry count stays the caller's delays, so a remote that lost objects outright still fails after the last attempt.
   Update the doc comments of `retryablePushRefusal` and `Repo.PushRebaseFree` (its "(failed)" or "cannot lock ref" sentence) and the retryable-reasons sentence in `internal/gitrepo/doc.go` to name the third reason.
2. `Repo.DeleteRemoteBranchLeased`, when the push's stderr carries `leaseRejectedTrigger` ("stale info"), returns `fmt.Errorf("gitrepo: git push --delete (leased at %s): %w", expectSHA, ErrLeaseRejected)`: the same text as today's wrapped error, now matching `ErrLeaseRejected` under `errors.Is`.
   Every other failure keeps today's wrapped git error and never matches the sentinel.
   Update `ErrLeaseRejected`'s doc comment to name both callers.
   `deleteRemoteBranch` in `internal/fabricengine/destroy.go` returns the error with `%w` kept, so a caller there tells a moved branch from a failed delete with `errors.Is`; the texts of `DiscardTaskBranch`'s and `Add`'s step-11 errors that wrap it keep their `leased at <sha>` detail.

Tests:
- `TestRetryablePushRefusal_ClassifiesByTheReasonOnThePushedRefsLine` gains a row whose stderr holds `! [remote rejected] refs/tags/archive/s/abc -> refs/tags/archive/s/abc (missing necessary objects)` for that ref and reads retryable, and a row with the same reason on another ref's line that reads not retryable.
- `TestPushWithRetry_RetriesOnlyRetryableRefusalsUpToTheBound` gains a row whose fake push fails once with that line and then succeeds, asserting two attempts and a nil error.
- `TestDeleteRemoteBranch` gains a row deleting under a lease on a SHA the bare remote's branch no longer sits at, asserting `errors.Is(err, gitrepo.ErrLeaseRejected)`, the text still naming `leased at <sha>`, and the branch still on the remote.

**ImpactSummary:** Widens one private predicate used by every `PushWithRetry` caller (archive-tag and plain pushes) and lets one public method's lease failure match a sentinel with its text unchanged; its only caller, fabricengine's `deleteRemoteBranch`, passes the error through, so no caller's text changes.

**Commit:** `1: gitrepo retries missing objects and names a lost delete lease`
