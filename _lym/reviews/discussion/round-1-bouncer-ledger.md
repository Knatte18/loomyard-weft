---
round: 1
ledger:
  - key: d4-memory-quotient-no-floor
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d3-prebuilt-lym-reaches-no-test
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d4-two-tier-concurrency-open
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-precheck-order-and-dropped-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-bound-misstates-skipped-features
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-package-set-misses-deleted-tests
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d9-retargets-one-of-three-refusals
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d6-nested-package-files-counted
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-contradicts-d6-on-maps-written
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-slot-background-does-not-admit-map-run
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-rerun-list-leaves-no-trace
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first ledger.
Two BLOCKING design findings stand: D4's memory quotient has no floor (F1), and D3's prebuilt `lym` reaches no test the call runs (F2).
Four MEDIUM design findings stand: the two tiers' concurrency (F3), D6's check order and dropped bound (F4), D1's skipped-feature Bound (F5), and D1's package set missing a package whose tests were deleted (F6).
Five LOW findings are local corrections whose text the findings already fix (F7 to F11).
