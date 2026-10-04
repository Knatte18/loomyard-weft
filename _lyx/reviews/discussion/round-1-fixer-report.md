# Round 1 fixer report

All edits are in `_lyx/discussion/decision-record.md`.

- F1: D6 now defines the fork point as the commit holding the origin record, skips the carry for a pair with no such commit, counts a parent-side delete as a change, treats identical changes as no conflict, and carries a task delete.
  Acceptance criteria gain the task-delete, identical-change, no-origin-commit and Publish cases.
- F2: the D2 checker reports an empty or whitespace-only overview; D4's bound points at it; the fixture list in Acceptance criteria includes it.
- F3: D9 lists the durable-content moves and the inbound-link files.
- F4: Open risks gains the pre-`aa67fd1d7` item, and D14's bound states the live-seed behaviour.
- F5: D2 states that the failure in an earlier-forked pair is by design and its message names the way forward; the Open risks line points at it.
- F6: the rejected-alternative sentence is removed from D13.
- F7: D13 states that the emptied anchor directory stays.
- F8: Acceptance criteria gain the webster template-test and landingshed fixture lines.

## Deferred

Nothing.
