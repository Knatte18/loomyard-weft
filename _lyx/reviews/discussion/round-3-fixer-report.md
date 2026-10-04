# Round 3 fixer report

## Fixed

- F1: D10's fold now adds only configured GitHub labels the target does not already carry, never a type label to a note, and states that D3 still refuses a target holding an orphaned label, with the way forward.
  An acceptance criterion covers it.
- F2: `intake close` takes an optional `completed` flag, and an `import` GitHub failure reports the exact `close` payload to run.
  The acceptance criteria name it.
- F3: D10 Sequencing now puts the recorded-issue, slug and `into` checks and the board write under one lock acquisition.
- F4: D9 requires positive-integer numbers and states its bound.
- F5: D10 `close` refuses an already closed issue, and an issue with two configured type labels is refused naming them.
  Both have acceptance criteria.
- F6: `support-log.md` Interview item 6 and Q9 now describe the `close` recovery.

## Deferred

None.

## Notes

- `close`'s `completed` flag is a new flag, bounded in D10: it only changes the close state, and `close` still writes nothing to the board.
- The mechanical gate `lyx loom validate-discussion` was not run in this round.
