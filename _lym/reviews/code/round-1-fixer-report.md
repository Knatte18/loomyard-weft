# Fixer-report, round 1 (code)

## Changes

All three findings are fixed as proposed, in one commit (`6159eac4`).
One commit rather than one per finding: F1, F2 and F3 edit the same two files, and `buttonclick_test.go` changes serve F1 and F2 together.

- **F1 (LOW):** `runButtonVia` in `internal/reedengine/buttonclick.go` stats `runDir` after `anchorOf` returns.
  A missing or non-directory `runDir` shows `lym: <label>: the anchor directory <runDir> is gone; point the worktree's anchor at an existing directory` and returns before the hash check, the confirm prompt and the run.
  The function's doc comment lists the new case.
  `TestRunButtonVia` gains an `anchorGone` field and the row `AnchorDirectoryThatIsGoneRunsNothingAndRaisesNoPrompt` (a confirm entry, so a missing check would raise a prompt).
  The `anchorMoved` rows now create the moved anchor directory on disk, since the check requires it.
  The expected message takes an `ANCHOR` placeholder beside `DIR`.
- **F2 (LOW):** the row `ConfirmEntryWithAnUnresolvableExecutableRaisesNothing` now sets `anchorMoved` and wants `ANCHOR/.lym/logs`, so reverting the message to `spec.Dir` fails it.
  The unmoved variant is not kept as a second row: it is the same behavior.
- **F3 (NIT):** the Info log text no longer claims the rendered directory is not the anchor.
  It reads `a button runs in its worktree's current anchor, which differs from the rendered run directory`, taking the reviewer's reword option.

Gate: `lym gate test ./internal/reedengine` passed (tier0, 1028 passed, 0 failed).

## Self-check

- fix-code-correctness: no findings.
- fix-consistency: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

Every fork returned a report; none is missing.

## Disputed

None.

## Deferred

None.
