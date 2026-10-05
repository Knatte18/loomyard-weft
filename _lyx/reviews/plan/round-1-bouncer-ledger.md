---
round: 1
ledger:
  - key: card1-halt-fixture-filed-lock-build-break
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card6-drop-rule-missing-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card4-tests-overlap-existing-voucher-tests
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card5-stated-bounds-not-pinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card3-grep-misses-stale-handoff-marker-prose
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first review, so no earlier ledger exists.
Five findings are open: two gating design findings (card 6's unbounded drop rule, card 4's test overlap with a voucher test card 1 deletes) and three non-gating ones (card 1's halt fixture build break, card 5's unpinned bounds, card 3's grep gap).
