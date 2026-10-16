---
round: 1
ledger:
  - key: d4-close-noise-fold-overlap
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: empty-inbox-shape-mismatch
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: close-rerun-no-bound-or-failure-route
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: upsert-label-write-unconfined
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: doc-go-in-item-no-criterion
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, none BLOCKING and one MEDIUM (F1, consistency).
F1 maps to `d4-close-noise-fold-overlap`, F2 to `empty-inbox-shape-mismatch`, F3 to `close-rerun-no-bound-or-failure-route`, F4 to `upsert-label-write-unconfined` and F5 to `doc-go-in-item-no-criterion`.
There was no previous ledger.
