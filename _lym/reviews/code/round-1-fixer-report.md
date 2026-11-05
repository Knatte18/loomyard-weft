# Fixer-report, code round 1

## Changes

One commit holds the round (`sketch: fix F1-F7`), since F1, F2 and F4 share the files F3, F5, F6 and F7 test; a second commit holds the self-check follow-ups.

- **F1** fixed as proposed. `endsAsSketch` in `internal/boardengine/sketch.go` takes the vocabulary and `removedDone` and passes only when `scopeChanged(sketchOf(before, vocab), now, removedDone)` is false.
  `checkRunLocks` takes the vocabulary; `board.go` passes `b.vocab` and both `replay.go` calls pass `*r.store.vocab`.
  `runlock.go` and `internal/boardengine/doc.go` word the net-change rule.
- **F2** fixed as proposed. `legacyStatusDiscussed` is the exported `LegacyStatusDiscussed`; the start guard refuses draft and discussed again, its comments and `releaseCommandText` name both, and the deleted test row is back.
- **F3** fixed as proposed. `TestWiredCommitStatus_FinishedRunEntry` has a `RefusedSketchLeavesTheEntryDraft` row with an entry another entry depends on; both wirings pass at tier 1.
- **F4** fixed as proposed. `MakeSketch` wraps the `validateWrite` refusal in the new `ErrSketchRefused`; `sketchFinish` falls back to draft only on it and returns any other error. The `sketchFinish` comment and `loomcli/doc.go` say so.
- **F5** fixed as proposed. A depended-on discussed task keeps its status in `TestDiscussedTaskBecomesASketchOnTheNextWrite`, `AbsentAndDoneEntriesStay` has an already-a-Sketch entry, and `TestFabricReplay` has a `HeldTaskBecomingASketchIsNotRefused` row.
- **F6** fixed as proposed. `TestRunLock_SketchExemptionIsTheWholeNetChange` in `sketch_test.go` has rows for a plain Sketch, a title change, a body change and a note without the sketch label.
- **F7** fixed as proposed. The `HeldTaskBecomesASketch` subtest sets and asserts `isolated`, `profile`, `pull_request`, `notify_on_land`, `project`, `brief`, `body` and `issues`.

## Self-check

- fix-code-correctness, LOW: the `NoteWithoutTheSketchLabelIsRefused` row passes with the old `endsAsSketch` too. Disputed, every fix worse (see Disputed).
- fix-proposal, LOW: the exemption text says "exactly" though `scopeChanged` ignores profile, run settings and the landing notice. Fixed: `endsAsSketch`, `checkRunLocks` and `doc.go` now say "no other scope change".
- fix-proposal, LOW: the same label row as the first line. Merged with it.
- fix-affected-set, LOW: the `set-status` help in `internal/boardcli/cli.go` said only a draft task is released, though the guard still refuses discussed. Fixed: both help texts name a task an older binary left discussed.
- fix-affected-set, LOW: a discussed task the migration cannot convert is layered as Ready, since the Discussed layer is gone. Deferred (see Deferred).
- fix-consistency: no findings.

## Disputed

- **Ground `every fix worse`, the label row of F6:** dropping `NoteWithoutTheSketchLabelIsRefused` leaves the boundary of a held task becoming a note without the `sketch` label unpinned, and the review asked for that row; the row guards a behaviour the new `endsAsSketch` still has to keep, though it also passed before.

## Deferred

- **The Discussed layer of `internal/boardengine/layer.go`:** restoring it reverses the retirement of the layer that the Sketch change made, which the review and the Goal did not ask for; whether a discussed task the migration cannot convert should show as its own README layer is the operator's call.
