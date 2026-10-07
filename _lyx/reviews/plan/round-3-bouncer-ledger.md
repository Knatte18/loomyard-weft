---
round: 3
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
    status: open
    class: consistency
    severity: MEDIUM
  - key: card-11-scope-omits-helpers
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: card-11-stop-leaves-card-12-free
    rounds: [3]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card-11-pins-run-snapshot
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised seven findings; all but card 11's premise were resolved by round 2.
Round 3 reads the card 11 premise as correct on the tree, so `card-11-false-premise` is resolved.
Card 11 still gates through a different defect: its target and stop rule do not cover the `internal/reedcli` test helpers its own premise names (F1, `card-11-scope-omits-helpers`).
Card 11's stop branch also leaves card 12 free to set `publish_verify` (F2) and pins a run snapshot (F4).
Card 9's Edit list, raised LOW in round 2, is raised again at MEDIUM (F3): it is not stated as a candidate set.
