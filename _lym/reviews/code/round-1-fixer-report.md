# Round 1 fixer report

All eight findings fixed. `lym gate test ./internal/loomcli ./internal/discussionparser` passed after the fixes and again after the self-check changes.

## Changes

- **F1** (`internal/loomcli/decisionreplace_test.go`, `decision_test.go`): added refusal rows `RecordWriteRunRecordNotCommitted` and `RecordFileAbsent`.
  Added commit-outcome rows for the unreadable HEAD, the `headCarries` error and the failed restore.
  The fake gained `headCarriesErr`, `restoreErr` and `recordWrites`, and its HEAD read now fails after a replace commit as well as after a commit.
- **F2**: extracted `boardRecordStoredOf` in `decision.go`, wired it for `boardRecordStored`, deleted the vacuous `AddOnABeamRun` row and added `TestBoardRecordStoredOf`, whose beam row fails when the `project` branch is removed.
  Fixed differently from the proposal: the proposal's first option (run the real closure in the fake) needs a board, so the seam is extracted and tested directly, the second option's shape.
- **F3**: rewrote the store `Long` text to the verb's admission and refusals.
- **F4**: added a sentence to the spec naming `lym board upsert --record-file` for the stored record.
- **F5**: `CheckAddedEntriesSubset` matches each current entry once and its doc comment says the whitespace trim applies; added duplicate-entry rows.
- **F6**: the overview line, the project-record stencil and the pawl recipe comment name `replace` beside `add`.
- **F7**: added `TestDecisionStoreVerb_TakesTheDecisionLock` with a success row and a lock-failure row.
- **F8**: extracted `recordHistoryFrom` in `projectrecord.go`, taking the trailers and two readers; `TestRecordHistoryCut` calls it, so the no-cut fallback to the whole history is reached through it.

A gate finding then flagged two comment lines of 120 characters or more, on `boardRecordStoredOf` and `recordHistoryFrom`.
The first split left a sentence of the `recordHistoryFrom` comment wrapped across two lines, which the gate flagged again.
That comment is now two sentences of one line each, and the `boardRecordStoredOf` one likewise.
The tests are rerun after the repair.

Commits: one per finding for F5, F4, F6 and F8; F1, F2, F3 and F7 share one commit because they touch the same files; one more for the self-check changes.

## Disputed

None.

## Deferred

None.

## Rulings

None.

## Self-check

Lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

- fix-code-correctness: no findings.
- fix-consistency, LOW: the `decision` group help names only `add` for a beam's scope. Fixed as proposed; the help names `replace` too.
- fix-consistency, LOW: the F4 sentence's "not with `replace`" is too absolute. Fixed with the same change as the fix-proposal MEDIUM: the clause is dropped.
- fix-proposal, MEDIUM: F4's "not with `replace`" is wrong for a promoted-Sketch run, where `replace` refreshes the stored copy. Fixed as proposed; the clause is dropped.
- fix-proposal, LOW: the `replace` help said "byte for byte" while the check trims trailing whitespace. Fixed; the help states the trim.
- fix-proposal, LOW: the F8 test inferred a cut from history length. Fixed; the test reads whether the whole-history fallback ran.
- fix-affected-set: no findings. It left F1, F7 and F8 unchecked, which are test additions.
