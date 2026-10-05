# Round 1 fixer report

All findings fixed in `_lyx/discussion/decision-record.md`; `support-log.md` needed no change.

## Changed

- F1: Goal, D9, D10 and Acceptance criteria name the interactive discussion role as the one exception to the operator-ban; its blocked steps still go to the parent.
- F2: the acceptance criterion now says a restart never loses an unconfirmed step and only a load past the timeout is skipped.
- F3: D6 gained a bound (abandon within the handoff timeout, no clear or compact without a passed gate, no marker left without a live watcher), mirrored in Acceptance criteria.
- F4: D1 states what a skipped skill lets through and that only the log shows it.
- F5: D9 defines the opening stencil per role and a registry mapping that D10's test reads.
- F6: D9 states that the sender is matched against the rendered parent name.
- F7: removed the dated history and the board-naming sentence from the D5, D7 and D9 rationales.

## Deferred

None.
