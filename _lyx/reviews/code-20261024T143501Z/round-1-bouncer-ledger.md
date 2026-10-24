---
round: 1
ledger:
  - key: vet-run-green-record-carry-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: vet-tier-list-exclusion-and-continue-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: tagtiers-sort-ints-vs-slices
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two LOW test-coverage findings on the vet run (green-record carry; tier-list exclusion and continue-past-failure) and one NIT on `sort.Ints` versus `slices.Sort`.
None is gating.
