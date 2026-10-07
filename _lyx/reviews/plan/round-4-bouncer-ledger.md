---
round: 4
ledger:
  - key: guard-keep-marker-coexistence
    rounds: [1]
    status: resolved
  - key: card-11-false-premise
    rounds: [1, 2]
    status: resolved
  - key: card-7-arguments-contradiction
    rounds: [1]
    status: resolved
  - key: sandbox-coverage-uncarried
    rounds: [1]
    status: resolved
  - key: gate-parity-pairs-list-missing
    rounds: [1]
    status: resolved
  - key: card-10-bound-omits-compile-only-tiers
    rounds: [1]
    status: resolved
  - key: card-12-bound-omits-publish-only-consequence
    rounds: [1]
    status: resolved
  - key: card-9-edit-list-overstated
    rounds: [2, 3]
    status: resolved
  - key: card-11-scope-omits-helpers
    rounds: [3]
    status: resolved
  - key: card-11-stop-leaves-card-12-free
    rounds: [3]
    status: resolved
  - key: card-11-pins-run-snapshot
    rounds: [3, 4]
    status: open
    class: consistency
    severity: LOW
  - key: card-9-candidate-set-misses-enforcers
    rounds: [4]
    status: open
    class: scope
    severity: MEDIUM
  - key: card-9-testtiming-doc-uncovered
    rounds: [4]
    status: open
    class: scope
    severity: LOW
  - key: card-index-11-overstates-card
    rounds: [4]
    status: open
    class: consistency
    severity: NIT
---

Rounds 1 to 3 closed the card 7, card 10, card 12, sandbox-coverage and gate-parity findings.
Round 4 shows the card 9 framing as a candidate set and card 11's scope and stop rule as sound, so `card-9-edit-list-overstated`, `card-11-scope-omits-helpers` and `card-11-stop-leaves-card-12-free` are resolved.
Card 11's Intent still records a point-in-time measurement (F3, `card-11-pins-run-snapshot`), now at LOW.
The remaining findings are new and narrow: card 9's candidate set omits two stencil scan enforcers (F1), `cmd/testtiming/main.go` documents the rule card 9 changes (F2), and the Card Index line for card 11 overstates the card (F4).
No open finding is design-class.
