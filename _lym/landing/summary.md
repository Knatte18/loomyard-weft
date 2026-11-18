# fabricengine: stop reading the prime records branch as ended on origin

A newly started batten run could fail at Seed-IDE with `the entry of branch main-weft is ended on origin, so the branch was not pushed and is left to its teardown`.
The branch is the prime records branch `main-weft`, which is never ended.
A rerun passed.

`PushPairAnchored` read any records side still unpushed after its push as an ended entry.
A commit landing on the records worktree while the push ran, or a tracking ref the push left unmoved, produced the same state, so the push was misclassified and the run failed.

## What changes

`recordsSideNotLanded` in `internal/fabricengine/pushanchored.go` now looks the branch up on origin before it classifies:

- A branch origin still holds returns an error wrapping `errRecordsUnpushedPastPush`.
  `PushPairAnchored` pushes the records side again over the push backoff (`gitrepo.PushBackoffDelays`), sleeping through `SyncOptions.BackoffSleep`.
- A records side still unpushed once the backoff is spent records a failed push of class `PushFailureDeferred`.
- A lookup that cannot reach origin records one of class `PushFailureTransport`.
- Both are returned as a `*PushFailedError`, so `IsPushOwed` is true and a run's transition retries the push instead of failing.
- Only a records branch gone from origin reads as one whose entry ended.
- Any other failed lookup returns its error unclassified.

`recordOwedRecordsPush` writes the failed-push record and mails the orch through the existing notifier.
`SyncOptions.BackoffSleep` now documents the backoff between a push's retries, and `doc.go` describes the new behaviour.

## Tests

`pushanchored_integration_test.go` gains a `post-receive` hook helper that commits to the prime's records worktree during the push.
It covers three cases:

- a commit during the push is pushed again and lands, with one backoff sleep and no failed-push record;
- a branch that keeps gaining commits is owed as `PushFailureDeferred` once the backoff is spent, with one sleep per delay;
- an origin lookup that cannot reach origin is owed as `PushFailureTransport`, with no sleep.

It also removes a duplicated comment line from the existing test's doc.

## To check by hand

- `records-push-retry-failure-paths-untested` (design, MEDIUM): the records re-push's failure paths in `PushPairAnchored` were left open by the last review.
  The new `PrimeRecordsPushOwed_*` rows cover the spent backoff and the transport lookup failure; check that no path is left untested, such as an unclassified lookup error or an unreadable branch in `recordOwedRecordsPush`.
- `records-push-transient-not-owed` (design, MEDIUM): check in `recordsSideNotLanded` and the retry loop in `PushPairAnchored` that every transient state of the records side ends as an owed push and none as a hard failure.

No regression check ran for this change, and the integration tests were not run by hand against a live batten run.
