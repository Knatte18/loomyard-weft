```yaml
lens: fix-consistency
outcome: reported
```

Task: check that the fixer's change set agrees with the sibling decisions, criteria and cross-references it touches.

Findings:

1. **MEDIUM.** The new D8 text says a recorded pull request merged at another head "is dropped from D4, committed, and Finalize lands the beam as with none recorded". This contradicts D4.
   - **Summary:** D4 says Finalize drops a pull request only after the parent-side landing. D8 now drops it before that landing.
   - **Evidence:** D4 says "Finalize reads it … and drops the pull request from it, committed, only after the parent-side landing completes." D4's acceptance criterion says "Finalize drops the pull request only after the parent-side landing." Neither was changed to allow the merged-at-another-head case.

2. **LOW.** D13's new bullet says the move "commits the carry into D4's record and the ready record's removal together in the beam worktree, as `lym beam reopen` commits the landing directory today".
   - **Summary:** The precedent it cites is a verb that D10 of the same record removes.
   - **Evidence:** D10 says "`lym beam reopen`, the reopen request, Beam-Await's handling of it … go". A reader after D10 lands finds no such verb. Naming the mechanism instead (`fabricengine.CommitAnchoredPaths` over the landing directory) would not go stale.

3. **LOW.** D13's new paragraph says "A pawl run found at Finalize with an open pull request in its scratch record has that pull request carried into D4's record once".
   - **Summary:** The record never says which step triggers this carry. The move runs only on a retired row, and the Bound and the Notes for the plan writer still say only retired rows trigger anything.
   - **Evidence:** D13's Bound says "only a status naming a row the recipe lists as retired moves". The Notes say "D13's retired-row map … the pawl side effects (session end, record carry, file removals, notice) hang on a seam the pawl wiring tells". A run at Finalize is not on a retired row, so nothing stated reaches this carry.

No further tool calls were made.
