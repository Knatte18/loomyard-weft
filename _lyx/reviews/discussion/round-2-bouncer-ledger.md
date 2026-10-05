---
round: 2
ledger:
  - key: d5-plan-review-unbounded-finding-edge
    rounds: [1]
    status: resolved
  - key: d11-file-level-parallel-comment-unbounded
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: acceptance-removed-phrases-incomplete
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: d13-split-not-carried-through
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: d5-webster-review-bullet-order
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: d1-bullet-absent-from-background
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 findings recur in round 2 as F1 (D11 bound), F2 (removed phrases) and F4 (D13, plus D1, D10, D11, D12 read as to-dos).
The round-1 D5 edge finding did not reappear; round 2's D5 finding F3 is a different issue, filed as `d5-webster-review-bullet-order`.
F5 is new.
Only F1 is gating.
