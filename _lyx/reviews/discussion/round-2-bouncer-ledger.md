---
round: 2
ledger:
  - key: d5-dead-pane-premise-vs-d6-reconcile
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
  - key: d12-count-bound-fails-in-prime
    rounds: [1]
    status: resolved
  - key: d3-window-split-validation-and-share-max-range
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-order-independence-unbounded-paths
    rounds: [1, 2]
    status: open
    class: decision
    severity: LOW
  - key: watchdog-constraint-unresolved-premise
    rounds: [1]
    status: resolved
  - key: d12-resume-teardown-acceptance-gap
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
---

Round 1 raised six findings; round 2 raised four, all mapped onto round 1 keys.

- The d5 key is the live blocker: a dead own-window strand is kept bound, but shuttle's stop, step stop and supersession skip non-live strands, so only a pair teardown removes it and the prime leaks windows.
- The d3 key now concerns `remove` failing on an invalid `window_split`, which blocks shuttle cleanup, seat stops and D12 self-removal.
- The d2 key now concerns whether a dead-pane strand counts toward a group's live count and when the group's recorded choice and size are dropped.
- The d12-resume key now concerns the failed chained self-removal leaving a dead bound pane that Resume relaunches.
- The d12 count-bound and watchdog keys were not raised in round 2 and are resolved.
