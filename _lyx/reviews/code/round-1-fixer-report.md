# Fixer-report, code round 1

## Changes

- F1, fixed as proposed: `TestTurnEndUsageLimited` gained the row `[earlierTurnEnd, limitTurnEnd, agentTurnEnd]`, no fork, want false (`internal/shuttleengine/claudeengine/activity_test.go`).
- F2, fixed as proposed: new `sidechainLimit` constant and the row `[earlierTurnEnd, agentTurnEnd, sidechainLimit]`, no fork, want false (same file).
  F1 and F2 share one commit, because both edit the same table.
- F3, fixed as proposed, then corrected by the self-check: `Run.holdNoticePending` (`internal/shuttleengine/hold.go`) gates the `usageLimited` case in the `Wait` switch (`internal/shuttleengine/wait.go`).
  The guard is false for a run with no notifier, an interactive run or a quiet-hold run.
  For a run with no `HoldDeferral` it is false while a pause covers the run.
  For a deferred-hold run it is false once a turn start was read past the turn end.
  `notifyHeld` filters interactive and quiet-hold runs, which I confirmed before choosing.

## Self-check

- fix-code-correctness, MEDIUM: a covered `HoldDeferral` run skipped the limit case, queued the hold and sent the stale notice after the pause lifted. Fixed: `covered` now counts only for a run with no `HoldDeferral`.
- fix-affected-set, MEDIUM: the same finding as above. Merged and fixed.
- fix-proposal, MEDIUM: the same finding as above. Merged and fixed.
- fix-proposal, LOW: `holdNoticePending` repeats the skip conditions of `notifyHeld`, so the two can drift. Not changed: the review asked for exactly this scoping, and sharing one predicate would also have to cover `notifyHeld`'s offset check and its persisting side effect.
  This is a design preference, not a defect, so it is not disputed either.
- fix-consistency, LOW: the narrowed case and its log line agree, and no doc needs a change. Checked, no change needed.
- fix-consistency, LOW: the same duplication as above. Merged.
- F1 and F2 rows: no findings from any lens.

## Disputed

None.

## Deferred

None.
