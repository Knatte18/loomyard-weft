# Round 3 fixer report

## Fixed

- F1: card 17 now states that `lyx loom goto` and `lyx shed goto` leave the run paused, so the orch runs `lyx loom start` after a goto, and adds a must claim for it.
- F2: card 21's startup row replaces `minElapsed`/`maxElapsed` with one `elapsed` check of kind `atLeastTimeout` or `zero`, and states each row's spec timeout and `wantTyped`.
- F3: card 21's watcher table declares its full row (`unknown`, `advances`, `endTurns`, `final`), the tick sequence and both rows' values, and drops the fallback to two tests.
- F4: card 17's After landing step now names "the prime's own copies" of the friction notes and the run's drive reports, with `internal/shedrun#DriveReportsDir` added to `Uses`, no `_lyx` path spelled in the stencil, and the claim rewritten to that wording.
- F5: card 20 now gives the true reason `TestRunner_ContextTokens_Delegates` stays, and its `ImpactSummary` names both files and the fake state it removes.

## Deferred

None.
