---
verdict: APPROVED
findings:
  - id: F1
    severity: LOW
    class: scope
    origin: lens:code-tests
    location: internal/creelengine/deliver_test.go:1980
    summary: Row ARingAnsweredByAClaimIsNotFollowedByTheBell still posts two messages to alpha inside one settle trigger, the pattern the new settle comment says to avoid.
    fix: In that row, drop the early e.start(t) and post the two bigBody messages before it, then call settle(e, func() { e.start(t) }) as the blocked-chain row does. The row's SetTurn call stays before the start. Leave the three-strand post in OnlyAStrandMarkedCreelHooksOnAnUnpausedPaneGetsTheBell unless it is shown to wake alpha's worker twice.
---

### [LOW] A second HookWake row still posts several messages in one settle trigger

**Location:** internal/creelengine/deliver_test.go:1980

**Issue:** The decision record's D4 scopes the fix to rows whose one trigger wakes the worker more than once, and the new `settle` comment states the rule that such a row posts before the start.
`ARingAnsweredByAClaimIsNotFollowedByTheBell` posts two messages to alpha inside `settle`, so its wake can buffer and run a probe after `settle` returns, and the following `settle` at line 1985 can return on that trailing probe.
The row's later steps await the ring and step the clock, so it probably passes, and the record's acceptance names only the blocked-chain and Stop-cap rows; hence LOW.
The three-strand post at line 2064 spreads over several strands and was not shown to wake alpha's worker twice, so it is not raised.

## Rejected

- Lens code-tests rated the same issue MEDIUM consistency and bundled line 2064 with it: lowered to LOW and class scope, since acceptance names only two rows and line 2064 is unproven.
- Lens code-correctness noted it could not see the worker code behind `e.start`: not a defect; the commit's 0 of 900 runs and the unchanged production worker support the new rows.

The Stop block, the unchanged tool-boundary block, the seam change with every caller updated, the docs and the tests all match the decision record's D1 to D3 and their acceptance criteria.
The diff touches nothing the record puts out of scope, and the commit messages match the diff.
