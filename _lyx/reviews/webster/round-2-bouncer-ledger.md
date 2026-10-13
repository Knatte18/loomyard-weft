---
round: 2
ledger:
  - key: own-window-relaunch-leaves-dead-pane
    rounds: [1]
    status: resolved
  - key: placement-constants-mixed-iota
    rounds: [1]
    status: resolved
  - key: claudeengine-test-budget-row-short
    rounds: [2]
    status: open
    class: consistency
    severity: BLOCKING
---

Round 1 raised the relaunch dead-pane finding and the mixed-`iota` placement constants; the fixes landed and round 2 confirmed both as not defects.
Round 2 raised F1, mapped to `claudeengine-test-budget-row-short`: the merge-in brought a test into `internal/shuttleengine/claudeengine`, leaving its `test-budget.yaml` row one short.
