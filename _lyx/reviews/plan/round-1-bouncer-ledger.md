---
round: 1
ledger:
  - key: guard-keep-marker-coexistence
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: card-11-false-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: card-7-arguments-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: sandbox-coverage-uncarried
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: gate-parity-pairs-list-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card-10-bound-omits-compile-only-tiers
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: card-12-bound-omits-publish-only-consequence
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 is the first round, so there is no earlier ledger.
Keys map to review findings F1 to F7 in order.
The gating pair is `guard-keep-marker-coexistence` (F1) and `card-11-false-premise` (F2); the rest are LOW.
