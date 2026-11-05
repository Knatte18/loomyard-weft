# Round 2 fixer-report

## Changes

- F1 (MEDIUM), fixed as proposed, commit `08a48537`.
  `cardNotDoneFor` now sets `createNotDone` for `rename-not-done` as well as `create-not-done`.
  The `createNotDone` field comment, the `createTargetWayForward` and `failCardNotDone` doc comments and the `doc.go` sentence say the way forward covers a rename-not-done finding.
  `TestFailCardNotDone_WayForward` gained a `rename-not-done` row on `record-batch` and one on an exhausted batch.
- F2 (LOW), fixed as proposed, commit `08a48537`.
  `failCardNotDone` computes `exhausted` as `Recoveries >= maxRecoveries && !rendersNewAmendment(in.Batch)`.
  The test table gained an `amended` column and a row at `MaxRecoveries` with an unrendered `AmendedCards` entry that wants the amendment text.
  The `failCardNotDone` doc comment and `doc.go` state the narrowed condition.
- Self-check LOW (fix-consistency), fixed, a follow-up commit: the `TestFailCardNotDone_WayForward` doc comment names the rename and unrendered-amendment arms.

`lym gate test ./internal/websterengine` passed after the first commit (tier0: 1093 passed, 0 failed).

## Self-check

- fix-code-correctness, LOW, Rename old side gets create-target wording: deferred, see below.
- fix-consistency, LOW 1, same old-side wording: duplicate of the item above, deferred.
- fix-consistency, LOW 2, stale test comment: fixed.
- fix-proposal, MEDIUM, F1's single flag misleads on a Rename old-side finding: deferred, see below.
- fix-proposal, F2: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

- The Rename old-side wording (fix-code-correctness LOW, fix-consistency LOW 1, fix-proposal MEDIUM).
  Both sides report under the one check ID `rename-not-done`, and a finding carries no field that names its side except the free text of `Detail`.
  Telling them apart needs a structured side marker from `internal/planglyph` on `planindex.Finding`, or a split of the check ID, which changes a contract that other consumers read, beyond what this round's findings ask.
  Matching on the `Detail` text would couple the engine to a message string, which is worse.
  The review's `fix` key chose one flag for both sides deliberately, and an old-side finding previously fell to the same dead-end default after a recovery.
  The way forward still names `recover-batch` first on a first failure.
  Raise a follow-up to add a side marker to the done-check finding.
