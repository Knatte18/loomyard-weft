no findings

Traced one concrete input through each changed path, using the gate results already in the context (tier 0 and tier 1 pass):

- `awaitLoopProcessExitMark` and `loopProcessExitRecorded` (F4): the mark is checked up to ten times, 100ms apart; it returns at once on a recorded `process N exited` mark, a cancelled `ctx` returns silently, and the wait stays bounded at about 1s. The new subtest fails without the fix.
- `loopLogLinesOf` refactor: `loopExitEvidence` keeps its old output; its callers are unchanged.
- F9 condition: the `StatusMovedInHistoryOnly` row fails against the old condition; the `status_moved` assertion holds for every row.
- `StatusMovedToEmptyProducer` row: passes against the old code too, since the `!= ""` guard already existed; it pins existing behaviour, as F8 asked.
- F2 test: the unbuffered `received` channel makes the test fail if the registration or the handler goroutine is removed; the "not returned yet" check cannot race.
- F7 rows: each of a missing mark, a stray `OnStart` and a wrong code fails the start-failure row.
- F3 row: if `DetachedLoopID` ignored the keeper flag, the envelope would be printed instead of written to the loop envelope file, and the row fails.

Tool calls made: none.
