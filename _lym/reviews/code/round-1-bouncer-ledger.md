---
round: 1
ledger:
  - key: filing-mark-hub-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised one finding, F1: the production wiring `shedrun.FilingMarked(location, slug)` in `hubStartGuard` has no test.
Only a fake sets the seam in `TestStartGuard_Refuse`.
The fix adds a subtest to `TestBeamLanding_ReachesWorktreeCreateSkip` that checks the refusal before the mark is written and the pass after.
