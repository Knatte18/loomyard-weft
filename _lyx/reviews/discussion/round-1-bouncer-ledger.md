---
round: 1
ledger:
  - key: d5-dead-pane-premise-vs-d6-reconcile
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-count-bound-fails-in-prime
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d3-window-split-validation-and-share-max-range
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d2-order-independence-unbounded-paths
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: watchdog-constraint-unresolved-premise
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: d12-resume-teardown-acceptance-gap
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first review of the discussion pair.
The gating findings are F1 (d5-dead-pane-premise-vs-d6-reconcile), where D5 assumes a dead pane survives until its owner removes it while reconcile kills it at the next reed op, and F2 (d12-count-bound-fails-in-prime), where the teardown bound does not hold in the prime.
The remaining four are LOW: F3 validation of `window_split` and range of `window_share_max`, F4 two ways a group's live count or share choice drifts past D2's order-independence, F5 the watchdog constraint that the code already answers, F6 missing acceptance lines for D12's resume and teardown behavior.
