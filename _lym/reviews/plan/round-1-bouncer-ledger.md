---
round: 1
ledger:
  - key: card1-redundant-test-rows
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card1-bound-holds-only-on-loom-pairs
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card4-config-test-header-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: set-recovery-fails-one-fan-at-a-time
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised four findings, all LOW or NIT.
F1 (card 1 redundant test rows) maps to `card1-redundant-test-rows`.
F2 (card 1 bound holds only on loom pairs) maps to `card1-bound-holds-only-on-loom-pairs`.
F3 (card 4 stale test file header) maps to `card4-config-test-header-stale`.
F4 (`--set` recovery rolled back per fan) maps to `set-recovery-fails-one-fan-at-a-time`.
No gating finding, so the loop converged.
