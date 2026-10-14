# Card 1 — shed-bounce-routed-to

**Edit:**
- `internal/shedengine#Shed.stepLocked`
- `internal/shedengine#foldsInto`
- `internal/shedengine#HistoryEntry`
- `internal/shedengine#appendOrFold`
- `internal/shedengine#TestRun_StuckWithOnStuckTarget`
- `internal/shedengine#TestRun_StuckWithNoTarget`
- `internal/shedengine#TestAppendOrFold_Appends`
- `internal/shedengine#TestEpisodeStuckCount_FoldedEqualsUnfolded`
- `internal/shedengine#composeActivity`
- `internal/shedengine/doc.go`
- `//contracts/specs/loom-status-spec.md`

**Intent:** D4, the Shed half: when Shed takes a Stuck verdict's `OnStuck` edge it records the target on that Stuck history entry's `routed_to` field.

In `Shed.stepLocked`'s `outcome == Stuck` arm, decide the arm before the entry is appended: the halt arm (`output.Halt || def.OnStuck == ""`) and the budget-exhausted arm append the entry with `RoutedTo` empty, as today; the routing arm (the `default` that persists `def.OnStuck`) appends the entry with `RoutedTo: def.OnStuck`.
The `appendHistory` closure gains the routed-to value as its one argument, every other caller passing "".
The budget comparison keeps reading `st.History`, the slice read at step 1, so the count and its boundary are unchanged.
The stored outcome stays `stuck` (`shedengine.Stuck`), and `episodeStuckCount` keeps reading `RoutedTo` only on `OutcomeReroute` entries, so a history with `routed_to` on its Stuck entries counts exactly as one without.

`foldsInto` adds one condition: two budget-exempt Stucks fold only when their `RoutedTo` agree; `appendOrFold`'s doc comment names the field among the matched ones.
`HistoryEntry.RoutedTo`'s doc comment changes with the added meaning (PATTERN-tolerant-records): it names the row a reroute entry sent the run to, or the row a Stuck entry's `OnStuck` edge bounced it to; empty on every other entry, and on a Stuck entry written by an older binary, which reads as before.
`doc.go`'s routing paragraph says the same, and so does `contracts/specs/loom-status-spec.md`'s status schema comment, which today says only a reroute entry carries `routed_to`.
`composeActivity`'s doc comment, which says a bounce cannot be told from the persisted values, now says the routing call site still tells `routedTo` because the activity line is composed in the same persist, while the Stuck entry itself now records the target for readers such as the checklist.

Tests, rows in the existing tests:
- `TestRun_StuckWithOnStuckTarget` asserts the persisted Stuck entry carries `routed_to` equal to the row's `OnStuck` target and its outcome stays `stuck`.
- `TestRun_StuckWithNoTarget` asserts the halted Stuck entry carries no `routed_to`.
- `TestAppendOrFold_Appends` gains a row: two budget-exempt Stucks equal but for `RoutedTo` append instead of folding.
- `TestEpisodeStuckCount_FoldedEqualsUnfolded` gains a case: the same history with and without `routed_to` on its Stuck entries gives the same count.

**ImpactSummary:** One write site in stepLocked and the fold predicate in shedengine; the status file gains an optional existing field on Stuck entries, read by no Go decision but the fold and the checklist later.
