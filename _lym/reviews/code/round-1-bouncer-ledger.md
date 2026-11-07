---
round: 1
ledger:
  - key: housekeeping-fold-loses-nil-probe-packing-pin
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: arrange-precondition-checks-vacuous-on-empty-field
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised two findings, none carried from earlier rounds.
F1 (`housekeeping-fold-loses-nil-probe-packing-pin`) is the gating one: the folded gc-prune row runs on an installed empty probe, so no row asserts packing on the default nil probe.
F2 (`arrange-precondition-checks-vacuous-on-empty-field`) is a latent gap: the precondition checks skip when the fixture field is empty.
