# Push webster reset's records when its sync fails after the commit landed

`webster reset` commits the run's transition to the records side and then pushes that commit synchronously,
so a push that fails writes a failed-push record that holds the run.
When the fabric sync returned an error, the verb refused before the push.
That error can come after the records commit landed, when recording the commit in the correspondence index fails.
The commit was then left to the detached pusher's quiet retry:
a push that failed wrote no failed-push record, and nothing held the run.

## Change

- `internal/webstercli/reset.go`: when `fabricSync` reports the records commit landed and returns an error,
  the verb runs the synchronous records push before it refuses.
  The refusal carries `push_error` when that push fails.
  A sync that fails before any commit landed refuses as before, with no push.
- The push and its skip conditions move into one method, `pushResetRecords`, which both paths call.
  The success path behaves as before: a hand-over fails the verb, and any other failure lands in `push_error`.
- The verb's help text and the package doc say that a refusal whose records commit landed still pushes it.

## Design decisions

- The refusal stays a refusal: the sync failed, and its existing way forward still applies.
  Only the push and its failed-push record are added.
- A hand-over on the failure path is reported in `push_error` beside the sync refusal rather than replacing it,
  since the verb already refuses there.

## Tests

`TestResetCmd_RecordsTheRunTransitionAtTheNewTip` gains the row `SyncFailingAfterTheCommitLeavesAnOwedRecord`.
It puts a directory where the correspondence index file should be, so the records commit lands and recording it fails,
and points the records origin at a closed port.
It requires the sync refusal, a `push_error`, an owed failed-push record of the run, and the transition commit on the records side.
Before the fix the row fails, because no push ran and no owed record was written.

## Merge danger

**Door:** two-way
**Blast radius:** webster
