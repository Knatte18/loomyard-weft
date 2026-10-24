# webster: a batch-start reset over a dead recovery earns a retry

After `lyx webster reset --to batch-start --batch NN` over a batch whose recovery ended dead, Merriam now runs `lyx webster recover-batch NN` itself instead of writing `outcome: stuck` again.

The reset clears the batch's recovery count and `RecoveryStartSHA` but leaves the record terminal and dead.
`RecoveryRetry` required a non-empty start to run its commit check, so it always answered false for a reset batch.
`status` then reported `recovery_retry: false`, and each fresh Master followed the stencil's dead rung and ended the run stuck.
The workaround was to run `recover-batch` by hand from the task worktree.

Closes #863

## Changes

- `RecoveryRetry` (`internal/websterengine/recoverbatch.go`) returns true for a terminal dead recovery record at zero counted recoveries, with no commit check.
  A record at one counted recovery still needs a commit of its own since the start, and a record at the cap still gets false.
  The kind, terminal and dead checks come first, so a fork record, a non-terminal record and a stuck record at zero still get false.
- `existingReportRemedy` in `beginbatch.go` words the retry remedy as "since the batch earns another recovery", because "committed work of its own" no longer holds for the reset case.
- The Master stencil (`webster-template-master.md`) says `recovery_retry: true` also covers a batch whose count a reset cleared.
  It also drops "that second recovery is the last", which was wrong for a reset batch.
- The `doc.go` retry paragraph, the `recover-batch` long help and the `RecoveryStartSHA` field comment state the new rule.

## Design decisions

- The reset verb's own state writes are unchanged.
  A non-terminal recovery record would make `recover-batch` attach to the dead strand instead of spawning, and rewriting the digest status would falsify the batch's report.
  The fix is in the verdict, so `status`, `recover-batch`'s terminal envelope and `begin-batch`'s remedy all read the same rule from `RecoveryRetry`.
- The new edge is bounded.
  It lets through only a record at zero counted recoveries, and the retry it grants is a counted spawn, so the cap of two and the commit rule bound every later retry.
- A dead recovery whose spawn rendered an amendment counted nothing, so it also reads as retryable at zero.
  The existing uncounted-spawn rule already allows that re-run.

## To check by hand

- A Master session already running when the reset lands reads the verdict only on its next `status` read, so the orch still has to `lyx loom resume` after the reset.
  End-to-end behavior against a real dead batch was not run; the new rows in `TestRecoveryRetry`, `TestBeginBatch_Refusals` and `TestStatusCmd` cover the verdict, the remedy and the `status` field.
