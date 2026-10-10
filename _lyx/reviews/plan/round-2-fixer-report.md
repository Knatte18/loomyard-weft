# Plan round 2 — fixer report

## Fixed

- F1 (LOW, consistency): the premise holds.
  Card 6's `turnStartRead` paragraph gave two reference offsets, and with a hold on show and a batch of `START` then `STOP:question`, reading it as "past `heldOffset`" skips `beginHeldWait` on a real turn end.
  `_lyx/plan/06-held-wait-ends-at-turn-start.md` now names one reference: the tick's own held turn end when the tick returns one, else `heldOffset` while a held wait is on show.
  It also states the outcome for that shape: `turnStartRead` stays false, `eventsRead` ends the old hold and the new hold shows, matching `Wait`'s existing `endHeldWait` on `eventsRead` before `beginHeldWait` (`internal/shuttleengine/wait.go:252`).
  `TestRun_Wait_HeldTurnEnd` gains a third `sessionFakeEngine` row: a `STOP:question` holds, then a `START` and a second `STOP:question` in one step leave the marker held, and output plus `STOP:done` finish.
  The row is a coverage gap, so it is added as a row of the existing test, not a new top-level test, and the test-budget file stays untouched.

## Disputed

None.

## Deferred

None.
