# Push webster reset's records when its sync fails after the commit landed

`webster reset` commits the run's transition to the records side and then pushes it synchronously,
so a push that fails writes a failed-push record that holds the run.
When the fabric sync returned an error, the verb refused before that push,
even though the error can come after the records commit landed, as when recording it in the correspondence index fails.
The commit was then left to the detached pusher's quiet retry, which writes no owed record and does not hold the run.

## Change

- `internal/webstercli/reset.go`: when `fabricSync` returns an error and reports that the records commit landed,
  the verb runs the synchronous records push before it refuses.
  The refusal carries `push_error` when that push fails.
  A sync that fails with no commit landed refuses as before, with no push.
- The push and its skip conditions move into one method, `pushResetRecords`, which both paths call.
  The success path behaves as before.
- The verb's help text, the file and package docs and the `fabricSyncWayForward` comment describe the new behaviour.

## Design decisions

- The verb still refuses after a successful push, because the sync failed, and the sync's way forward still applies.
- A hand-over reported by the push on this path changes the refusal:
  the error names the hand-over first, keeps the sync error, and replaces the sync's way forward
  with one that leaves the run to the other machine.
  The sync's way forward would point at commits that nothing will push.
  `push_error` carries the hand-over too.

## Tests

- `TestResetCmd_RecordsTheRunTransitionAtTheNewTip` gains the row `SyncFailingAfterTheCommitLeavesAnOwedRecord`.
  A directory on the correspondence index path makes the sync fail after the commit lands,
  and the records origin points at a closed port.
  The row requires the sync refusal, a `push_error`, an owed failed-push record of the run
  and the transition commit on the records side.
- `TestResetCmd_SyncFailureAndTheRecordsPush` requires that a sync failing before any commit landed pushes nothing,
  and that a hand-over after the commit landed is named in the error and in `push_error`.

## To check by hand

- The merge danger of the added push in `internal/webstercli/reset.go`:
  the sync-failure path now pushes outward, and a failure there writes a failed-push record that holds the run.
  Confirm that the door and blast radius below account for both.
- The hand-over case is tested with a stubbed push only; no takeover by another machine was staged against an origin.

## Merge danger

**Door:** two-way
**Blast radius:** runs

The reset now pushes the pair to origin before it refuses, when its sync fails after the records commit landed.
That commit already starts a detached push of the same pair, which is why the door is called two-way.
A failed synchronous push writes a failed-push record, which holds the run.
