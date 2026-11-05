# Fixer-report — code review round 1

Gate: `lym gate test ./internal/websterengine ./internal/planglyph` passed at tier0 (1609 passed, 0 failed).

## Changes

- **F1 (fixed as proposed, with one extension).**
  `failCardNotDone` passes whether `in.Batch.Recoveries` has reached `maxRecoveries` to `createTargetWayForward`, which then returns the way forward `RecoverSpawnOrAttach` gives an exhausted batch.
  That text now lives in one helper, `exhaustedRecoveryWayForward`, which both callers use; `RecoverSpawnOrAttach`'s message is unchanged.
  Uncheckable records still take the fresh-restart steps first.
  Extension from the self-check: `deleteReferencedWayForward` and `glyphRejectedWayForward` take the same exhausted branch, since they end in the same dead end.
  `_lym/landing/summary.md`: deleted the two sentences about uncounted recoveries, added a bullet for the exhausted case, and reworded the first bullet to "Create target or Rename new side".
- **F2 (fixed as proposed).**
  New `internal/websterengine/export_test.go` exposes `failCardNotDone`, `cardNotDoneInputs`, `maxRecoveries` and a `CardNotDoneFor` wrapper over `cardNotDoneFor`.
  New table-driven `TestFailCardNotDone_WayForward` in `recordbatch_test.go` asserts the exact way-forward clause for: a signature-mismatch finding (built through `cardNotDoneFor`), a plain create-not-done, create-not-done after a recovery, an uncheckable record, exhausted batches (create, signature-mismatch, glyph-rejected, referenced delete), the glyph-over-create and delete-over-create precedence, and an unreferenced delete leaving create-not-done.
- **F3 (fixed as proposed).**
  `createTargetWayForward` words the amend step "the card's Create target or Rename new side".
  `TestDoneCheckVerdicts_NotFoundDetail` gained a `SignatureMismatchOnRenameNewSide` row.
- **F4 (fixed as proposed).**
  Rows for a target without a parameter list, another owner, another unit and a Go target, each expecting create-not-done.
- **F5 (fixed as proposed).**
  `TestDoneCheckVerdicts_NotFoundDetail` compares the whole detail with equality; the table builds the expected text with a `createDetail` helper.

Gate repair: the gate refused the round because two comments added here were wrapped mid-sentence, `failCardNotDone`'s doc comment in `recordbatch.go` and the `TestFailCardNotDone_WayForward` comment in `recordbatch_test.go`.
Both now break at sentence boundaries, one sentence per line, and the gate re-run on `./internal/websterengine` passes (1090 passed, 0 failed).

Commits: `e7f5cc82` (F1, F2), `49fdb35c` (F3, F4, F5), a follow-up for the self-check extension of F1, and one for the gate repair.
`_lym/landing/summary.md` is edited in the working tree and not committed, per the `_lym` rule.

## Self-check

- fix-proposal, MEDIUM: the exhausted steps never tell the operator to amend the card, so the Create dead end can recur one hop later. Deferred, see below.
- fix-consistency, LOW (F-A): the glyph-rejected and delete-reference way forwards keep the dead end. Fixed with an own change (exhausted branch added to both, two test rows).
- fix-affected-set, LOW: same finding as F-A, merged. Fixed the same way.
- fix-consistency, LOW (F-B): `summary.md` still said "Create target" only. Fixed.
- fix-code-correctness, LOW: row `GoTargetIsNotAMismatch` cannot reach the Go guard. Disputed, see below.
- fix-proposal, LOW: a plain `rename-not-done` finding gets the default recover-batch way forward. Not fixed: it predates this change and sets neither flag; the lens itself calls it an observation. Deferred, see below.
- No fork failed to return a report.
  The four lens reports are saved beside this file as `round-1-fixer-report.lens-<lens>.md`.

## Disputed

- **fix-code-correctness LOW, `GoTargetIsNotAMismatch`: ground `every fix worse`.**
  The premise holds: a Go glyph carries no parameter list, so no not_found Go key can reach the guard past `wanted.Params == nil`.
  Making the row stronger is impossible, and deleting it drops the Go negative case F4 asks for, so the row stays as a contract pin of the Go detail text.

## Deferred

- **fix-proposal MEDIUM, exhausted steps lack an amend step.**
  Whether `lym webster reset --to batch-start` accepts a plan edit and rebaseline before it re-drives the batch is not settled by the code I read, and the lens could not confirm it either.
  The way forward is the text `recover-batch` already gives an exhausted batch, as the review specified.
  Adding an amend step needs a decision on what the reset does with an edited plan, which is an operator or design call.
- **fix-proposal LOW, a plain Rename not-done finding has no way forward of its own.**
  It predates this change and the review did not ask for it; fixing it means a new check-flag path, which belongs in its own entry.
