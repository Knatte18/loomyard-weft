---
dispositions:
  - id: F1
    disposition: fixed
  - id: F2
    disposition: fixed
  - id: F3
    disposition: fixed
---

# Fixer report, round 1

## Changes

- F1 (`5cd3ce73`): `internal/fabricengine/pushanchored.go` now records both transient outcomes as owed pushes.
  A records side still unpushed once the backoff is spent records a `PushFailureDeferred` failed push.
  A transport failure of the post-push origin lookup in `recordsSideNotLanded` records a `PushFailureTransport` one.
  Both return the record's `*PushFailedError` through the new `recordOwedRecordsPush`, so `IsPushOwed` holds.
  A non-transport lookup error and the ended message stay unclassified.
  `recordsSideNotLanded` takes `SyncOptions` for the run slug and notifier.
  The `PushPairAnchored` and `recordsSideNotLanded` docs and the `doc.go` records-side lines say so.
- F2 (`3ade81ec`): `pushanchored_integration_test.go` gains the helper `installRecordsCommitHook` and the table `PrimeRecordsPushOwed_*`.
  Row `BackoffSpent` commits on every push and asserts a deferred `*PushFailedError`, `IsPushOwed`, `len(gitrepo.PushBackoffDelays())` sleeps and one deferred record.
  Row `OriginLookupTransportFailure` repoints origin at a refused URL after the first push and asserts the transport classification, owed, no sleeps and one transport record.
  `PrimeRecordsCommitDuringThePushIsPushedAgain` counts its sleeps and asserts one.
- F3 (`57807c50`): the `SyncOptions.BackoffSleep` doc in `fabric.go` now reads "sleeps one delay of the push backoff between a push's retries; nil means sleep."

A gate repair then reflowed two comments the round added, the `doc.go` records-side line and the `installRecordsCommitHook` doc.
The first became a single line and the second one sentence per line, with no change to what either says.

A second gate failure named `internal/websterengine`, a package this round never touched.
Its log shows the tmuxkit guard finding a `/tmp/.git` that appeared during the run, written by a test running beside it.
No change was made: `lym gate test --tier 1 ./internal/websterengine` passes on a re-run and `/tmp/.git` is gone.

`lym gate test --tier 1 ./internal/fabricengine` passed, with the new subtests among the passes.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings; it left unchecked whether other tests pass a no-op `BackoffSleep` into the records re-push, and nothing in the review's premise needs it.

## Disputed

None.

## Deferred

None.

## Rulings

None.
