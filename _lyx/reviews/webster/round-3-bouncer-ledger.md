---
round: 3
ledger:
  - key: own-window-relaunch-leaves-dead-pane
    rounds: [1]
    status: resolved
  - key: placement-constants-mixed-iota
    rounds: [1]
    status: resolved
  - key: claudeengine-test-budget-row-short
    rounds: [2]
    status: resolved
---

Round 1 raised the relaunch dead-pane finding and the mixed-`iota` placement constants; both were fixed.
Round 2 raised the `claudeengine` test-budget row one short after the merge-in; commit `54d23080e` set the row to match, and round 3 found nothing further.
