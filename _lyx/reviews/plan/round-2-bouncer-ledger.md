---
round: 2
ledger:
  - key: guard-keep-marker-coexistence
    rounds: [1]
    status: resolved
  - key: card-11-false-premise
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
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
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised seven findings, one BLOCKING and one MEDIUM design finding among them.
Round 2 no longer raises the marker-coexistence finding, and the review settles it, so it is resolved.
The other five round-1 LOW findings are not raised again and are resolved.

Card 11 still gates: its premise (a failure the tree does not reproduce, a cause site the evidence does not support, and no outcome for an already-green tier) is the round-2 form of the round-1 finding.
Card 9's Edit list is a new LOW consistency finding.
