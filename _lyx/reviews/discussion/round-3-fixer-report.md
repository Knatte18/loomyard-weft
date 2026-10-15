# Round 3 fixer report — discussion

## Changes

- F1, fixed with the fix key's alternative ("or give another rule").
  The Stops by settler Decision now matches by the record's last counted entry instead of counting entries around the change.
  A batten-file record names the batten-row halt whose own entry sits at its history length.
  A loom-file record names a child stop with a change when its producer and state are equal and its history length is the greatest among such records whose last counted loom entry is stamped at or before the change.
  An earlier record that a restep before batten's observation left behind folds into the same stop and adds no finding.
  A note-only stop is matched on producer and state alone and takes the earliest record no other stop took, falling to the later items when none is left.
  An empty producer on a change matches no record.
  I did not use the "count of entries stamped strictly before the next entry of either history after the change" rule, because it fails in the lag case it targets.
  When the driver resteps before batten observes, the restep's loom entry is stamped before the change, so the count still includes it and overshoots the record's length.
  Three Acceptance criteria lines are added: the lagged restep, the note-only stop with the empty producer, and `by` `shed`.
- F2: took the first option.
  A batten-row halt is now a batten history entry at a batten row other than `Run-Shed` that left batten blocked or awaiting, and the report counts it apart from `convergence`'s halts.
  The batten file's records name these halts.
  Waits now give a batten-row halt a wait from its own entry.
  The criteria line that was at 202 now says a batten-file record names only a batten-row halt, counted apart.
- F3: each interval now runs from the latest merged stamp so far, and an out-of-order stamp's successor starts from that latest stamp.
  The two criteria lines on intervals match this.
- F4: the first Notes bullet is reworded as proposed, and the Rationale now cites the 2026-10-12 computation's per-row split instead of its "interval rule".
- F5: settle files are read at the tag, or at the records branch tip for a run in flight; a record not yet committed there leaves its stop `open`.
- F6: item 6 reads the `by` label `shed` as class `shed` and every other label as its own value.

## Disputed

None.

## Deferred

None.
