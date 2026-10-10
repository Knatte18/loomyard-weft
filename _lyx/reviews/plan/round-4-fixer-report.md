# Round 4 fixer report — plan

## Fixed

### F1 — the teardown's mark is consumed by the first dead-loop stop

Validated against cards 12 and 18: `deadLoopStop` retired the pid file and wrote an envelope, so the second invocation delivered it and the third spawned a loop before `RemovePair`, against D11's bound "no step starts in a pair once its session-end step begins".
D11's bound is a decision, so I took the reviewer's first fix, not the narrowing.

- Card 12 creates `TeardownLoopID`.
  `runUntilStop` runs `deadLoopStop` over a free lock beside the mark whatever else the directory holds, and every waiter ends there whatever id it adopted.
  Over the mark, `deadLoopStop` writes `failed` under its bound, prints the `loop-exited` stop with `loop.detail` naming the teardown, writes no loop envelope and keeps the mark.
  The envelope-delivery retirement of a leftover pid file now retires only a pid file whose loop id equals the envelope's `loop_id`, and never the mark.
  A `--loop-detached` `runLoop` that finds the mark refuses to step, as over a dead loop's record.
  A stated bound: only card 18's seize writes the mark, and it puts back what the mark replaced when the session end fails.
- Card 18: once it holds the loop lock, `seizeLoopLock` replaces whatever pid file it finds with the mark, and leaves the mark after the session end.
  When `EndSession` fails before the session end, the release puts back the record it read, or removes the mark, so a pair whose removal failed is not barred from looping forever.
  This also closes the gap the review names between `killLoop` and the seize: a `deadLoopStop` there retires the killed loop's record, and the seize then writes the mark over nothing.
  Uses gains `TeardownLoopID`; the doc line is updated.
- Test rows (coverage gap, so rows on existing tests): card 12's `TestUntilStop_DeadLoopStopsLoopExited` gains a first, second and third invocation over the mark, a variant with an undelivered envelope beside the mark, and a waiter that read a held lock with no pid file stopping once the lock is released beside the mark.
  Card 18's release row now asserts the mark whether or not a killed loop's record was there, and the run-lock-timeout row asserts the pid file is left as the seize found it.

### F2 — a waiter that never read its loop's pid file misreads a later loop's records

Validated: with one shared delivered slot, a later loop's delivery replaced the record, so a waiter that never read its own pid file adopted the later loop or reported `loop-exited`.
I took the reviewer's first option: the delivered record is keyed by loop id.

- Card 3: `LoopDelivered(l, runID, loopID)` is `loop-envelope.<loopID>.delivered.json`, with a `paths_test.go` row that two loop ids differ.
- Card 15: `loopSpecFor` fills `Delivered` from `LoopDelivered` over the run-id.
- Card 12: `LoopSpec.DeliveredPath` becomes `Delivered func(loopID string) string`.
  A waiter prints its own loop's delivered record.
  Delivering an envelope renames it onto its own loop's record, and so does retiring a stale one undelivered, so a finished loop always leaves a file carrying its id.
  The card states that consequence, so the gone rules now see only a loop that never finished.
  The `busy` delivered-elsewhere rule, its error and its `loop delivered elsewhere` refusal row are removed, since that state can no longer arise.
  `TestUntilStop_SecondWaiterAdoptsTheLiveLoop`'s delivered-elsewhere row is corrected: in two rows the waiter sleeps through its loop's delivery and a later loop's run, once having read its pid file and once not.
  Both rows assert it prints its own loop's envelope, never the later loop's, with no `loop-exited` and no `failed` write.

## Disputed

None.

## Deferred

None.
