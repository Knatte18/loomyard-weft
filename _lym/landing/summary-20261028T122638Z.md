# Retry a rejected push over the backoff before reporting a divergence

Two pushes of one branch on one machine can race.
The loser's rejection, or a fetch its retry made while the winner held the tracking ref, then reported a `not_fast_forward` divergence that the winner's landing had already removed.
The orch got a notice with a `git merge` or `git reset --keep` way forward for a branch whose next push was a plain fast-forward.
A records notice also said "another machine wrote the same paths" with `paths: none`.

## What changes

`pushClassified` (`internal/fabricengine/pushclassified.go`) now runs a bounded retry, `settleRejection`, on any rejected push before it records a failure.
This covers the code, records and bolt sides.

- Each retry is the side's own retry.
  The records and bolt sides use the existing replay retries, `retryWeftPush` and `retryBoltPush`.
  The code side uses the new `repushOverOriginContained`.
  It fetches, and pushes again only while origin's tip is contained in the local tip.
  When origin holds commits the branch lacks, it stops at once and the rejection is reported with no wait.
- The retry runs again only after a bare rejection, or after an error no failure class names, such as a failed fetch.
  It stops on a landing, a deferral, a replay conflict, an anchor stop, an auth refusal or a transport failure.
- Attempts are spaced by `gitrepo.PushBackoffDelays()` and slept through `SyncOptions.BackoffSleep`, the same schedule a gone branch or an unreachable origin already waits out.
  A rejection that outlasts the backoff still writes one `not_fast_forward` record and one notice, as `PATTERN-push-both-sides` requires.
- `pushRebaseFreeRetryingOnce` is removed, because the code-side retry does what it did, repeated over the backoff.

The records and bolt way forward, `recordsConflictWayForward` (`internal/fabricengine/failedpush.go`), now says origin holds commits this branch lacks and never names a machine.
It names conflicting paths only when a replay stopped on some.
With none, it drops the `paths:` clause and says "settle any conflict" instead of "settle the named paths".

The push-classification section of the package doc (`internal/fabricengine/doc.go`) and the `PushAnchored` doc comment describe the new retry and wording.

## Design decisions

- **A notice names no machine.**
  The board entry allows naming another machine only when origin's commits carry its name.
  No commit lym writes carries a machine name, so that condition never holds, and the notice never names one.
- **A persisting rejection keeps the `not_fast_forward` class.**
  This holds even when origin is contained in the local tip, as with a declining hook.
  Reclassifying that case was not asked for.
- **The race's exact path is not pinned.**
  The fix re-checks before reporting, on every side, rather than changing which lock a given push takes.

## Tests

- `TestPushFailureClasses/APushThatLostARaceIsRetriedUntilItLandsWithNoRecord` has Code and Records rows.
  A hook declines the first push and the first retry.
  The push then lands after one backoff sleep, with no record and no mail.
- `RejectedBecauseTheOtherMachinePushedFirst` now also asserts that a code push meeting origin ahead sleeps no backoff.
- `ARecordsRetryWhoseSecondPushIsRejectedAgainWritesANotFastForwardRecord` asserts that the whole backoff is waited out before the record.
- The `TestFailedPushRecord` way-forward table has the new wording and a records row with no conflicting paths.
- Tests whose remote keeps declining now inject a no-op `BackoffSleep`, so they do not wait out the real backoff.
  These are in `pushanchored_integration_test.go` and `boltpullfirst_integration_test.go`.
