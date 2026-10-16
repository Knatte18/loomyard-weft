---
round: 1
ledger:
  - key: free-lock-vs-exiting-daemon
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d5-path-prepend-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: d8-creel-check-no-strand
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: goal-680-broader-than-scope
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: bounds-without-criteria
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none recurring.
The gating one is `d8-creel-check-no-strand` (F3): the creel suite only starts fresh-shell strands, so D8's reworded pane-shell check has nowhere to run.
`free-lock-vs-exiting-daemon` (F1) and `d5-path-prepend-unbounded` (F2) are MEDIUM but not gating-class, and their fixes are fully spelled out in the review.
`goal-680-broader-than-scope` (F4) and `bounds-without-criteria` (F5) are LOW.
