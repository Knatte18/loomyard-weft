# webster: a dead recovery a batch-start reset cleared earns a retry

Fixes #863.

When a batch's recovery ends dead with `recovery_retry: false`, lyx names `lyx webster reset --to batch-start --batch NN`, then `lyx webster recover-batch NN`, as the way forward.
The reset clears the batch's recovery count and `RecoveryStartSHA` but leaves its record terminal and dead.
`RecoveryRetry` returned false for any record with an empty recovery start, so after the reset `lyx webster status` still reported `recovery_retry: false`.
Each fresh Master read that verdict, followed the stencil's dead rung and wrote `outcome: stuck` again, and the orch had to run `recover-batch` by hand.

## The change

`RecoveryRetry` (`internal/websterengine/recoverbatch.go`) now reports true for a terminal dead recovery record with no counted recovery.
The batch's first counted recovery is still owed in that state, as it is for a fork that reported stuck, so no commit check applies.
A record with one counted recovery still earns a retry only when the recovery committed work of its own, and the cap of two is unchanged.

`status`, `recover-batch`'s terminal envelope and `begin-batch`'s report-present remedy all read `RecoveryRetry`, so all three now name `recover-batch` for the reset batch.
The begin-batch remedy's wording changed from "since the dead recovery committed work of its own" to "since the batch earns another recovery", because it now covers both cases.

## Design decisions

- The reset's own state writes are unchanged: the batch stays terminal and dead.
  A non-terminal recovery record would make `recover-batch` attach to the dead strand instead of spawning, and rewriting the status would contradict the batch's digest and report.
- A dead recovery that counted nothing because its prompt rendered a new amendment also reads as retryable now.
  The uncounted-spawn rule already allowed that re-run, and the retry is itself a counted spawn, so the cap and the commit rule still bound every later one.

## Docs and stencil

- The Master stencil's dead rung now says `recovery_retry: true` also covers a batch whose count a `reset --to batch-start` cleared, and no longer calls the retry "the last".
- `internal/websterengine/doc.go`, the `recover-batch` long help and the `RecoveryStartSHA` field comment describe the new rule.

## Tests

- `TestRecoveryRetry` gains a row for a reset-cleared dead record (true, zero recoveries).
- `TestBeginBatch`'s "terminal dead record ends the run stuck" row now has one counted recovery, so it still covers the no-retry wording.
  A new row covers the reset case naming `recover-batch` once more.
- `TestStatusCmd` gains a row in which a reset-cleared dead batch reports `recovery_retry: true`.
