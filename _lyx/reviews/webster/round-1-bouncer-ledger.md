---
round: 1
ledger:
  - key: merge-in-verb-spelled-three-places
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: synced-row-does-not-reach-intersection
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised two LOW consistency findings.
F1 maps to `merge-in-verb-spelled-three-places`, the parked record's verb declared in three places.
F2 maps to `synced-row-does-not-reach-intersection`, a test row that passes without the intersection.
Neither is gating.
