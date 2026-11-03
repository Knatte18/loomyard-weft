---
round: 2
focus:
  - "internal/battencli/dobbyreads.go, the DobbyRuns closure that sets ReachedParent: does a test in the repo reach it through production wiring, covering the paused-only gate, the argument pair passed to childReachedParent and the unreadable-judgment path?"
  - "internal/dobbyengine/passes.go, the liftPending cleanup condition (!drivable instead of State != running): does a test fail if the line is reverted, using a paused run with ReachedParent set rather than a blocked run?"
  - "internal/dobbyengine/passes_test.go TestPlanPass: is a paused ReachedParent run with OwnedElsewhere or Held set pinned, and do the added OriginOwed and CancelMarked rows duplicate running-run rows?"
  - "lym batten goto leaves a run paused with no pause entry: can dobby's reached-parent read resume such a run from child history alone, undoing the goto, and do contracts/orchrole/orch-role.md rule 3 and internal/beamcli/doc.go say what holds it?"
---

Focus lists places to check, not verdicts.
