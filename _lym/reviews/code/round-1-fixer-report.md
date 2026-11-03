## Changes

All five findings are fixed as proposed unless marked otherwise; every commit is individual and nothing is pushed.

- **F1** (`3ff20e3e`): `CaptureRecord` carries `Skip`, the tests the capture left out, and `RunCapture` records it.
  `RecordScope` compares the new skip list against the capture's own list through `dropCaptureSkipping`, not against the current scope record's.
  `TestRecordScope` gained a row for a same-skip replacement, the existing drop and keep rows seed a capture that records its skip list, and a chain subtest (partial skipping `a`, then off, then full) wants the capture removed.
  `TestRunCapture` asserts the recorded skip list.
  The `RecordScope` doc, `driftengine/doc.go` and the `lym drift scope` help say "the capture skipped".
- **F2** (`823d7a85`, plus the loomcli commit after `fc427cc6`):
  - `TestDrift_RunsInTheAnchoredDirectory` (tier 1, `internal/driftcli/anchor_integration_test.go`) runs the built `lym drift` in a hub anchored at `backend`.
    The main capture child, the compare and the side capture each record their `pwd`, and all must equal the anchored directory.
    A child started outside the anchored directory cannot resolve its worktree, so it never records its start and the test fails.
  - `TestDriftDeps_RunInTheAnchoredDirectory` (tier 1, `internal/loomcli/driftwiring_integration_test.go`) drives `RunCapture` and `Compare` of `driftDeps` over the same shape of hub.
  - The `StartCapture` seam of `driftDeps` spawns `os.Executable`, which a test cannot run, so its request is extracted to `captureSpawnRequest`.
    `TestCaptureSpawnRequest_StartsInTheAnchoredDirectory` (tier 0) asserts its `AnchorDir` is the anchored directory.
    This closes the self-check's one MEDIUM finding.
- **F3** (`3ff20e3e`, with F1): `dropCaptureSkipping` runs inside the locked scope update before the replacement is built, so a failed read or removal returns an error and the old scope stays.
  A subtest with an undroppable capture path asserts the scope record is unchanged.
- **F4** (`0e16d976`), fixed differently: the review proposed either carrying a separate worktree root or changing the `Holder.Worktree` doc comment.
  The doc comment now says it is the directory the holding run works in, the worktree root or the anchored directory inside it.
  I did not carry a worktree root field through `stepRun`, `CaptureRequest` and `CompareRequest`: every reader of the field only prints it, so a new field would add three request fields for nothing.
  I added no holder assertion, since the doc change fixes no behaviour.
- **F5** (`fc427cc6`): `startFailure` returns a `childNoRecordError`, and `captureAgain` turns only that into the capture halt.
  Other `StartCapture` errors, such as an unresolvable binary, an unwritable child log and a held capture lock, are returned as before.
  `TestDriftBaseline` gained rows for an environment fault and a held lock, and the child-no-record row uses the real `startFailure` error.

Gate: `lym gate test` tier 0 and `--tier 1` pass for `./internal/driftengine`, `./internal/driftcli` and `./internal/loomcli`; tier 0 passes for `./internal/gateslot`.

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal, LOW (F3): the capture is gone if the scope write itself fails after the drop. Disputed, `every fix worse`: dropping after the write is the F3 defect, and the loss costs a recapture, never a wrong compare.
- fix-proposal, LOW (F1): a capture record from a binary that predates `skip` reads as skipping nothing. Disputed, `every fix worse`: an absent key cannot be told from an empty list, and dropping every capture without the key would drop valid fresh ones; the record is ephemeral.
- fix-affected-set, MEDIUM (F2): the loomcli `StartCapture` seam was unpinned. Fixed with `captureSpawnRequest` and its tier 0 test.
- Every fork returned a report; none is missing.

## Disputed

- Self-check LOW on F3, ground `every fix worse`, evidence above.
- Self-check LOW on F1, ground `every fix worse`, evidence above.

## Deferred

Nothing deferred.
