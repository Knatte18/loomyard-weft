---
round: 1
ledger:
  - key: prerun-pull-first-drops-stencils-seed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: bolt-tests-not-on-hubforge
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: isnoserver-misses-absent-socket
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: drop-gate-binds-only-base-name
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: tracked-dirtiness-refuses-normal-state
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: push-drop-has-no-recorder
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: card7-repeats-issseedcommit-refuse-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: card7-bundles-three-units
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: kindcommitsdropped-missing-from-oracle
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: bolt-type-edit-target-no-change
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
  - key: card3-impact-claims-writer-keys-early
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eleven findings, none carried from an earlier round.
The two BLOCKING ones are the per-subtree pull-first that drops the stencils seed commit (F1) and Bolt tests that bypass `hubforge` (F2).
Card 7 draws most of the rest: the drop gate's weak bound, the tracked-dirtiness refusal, the missing recorder on the push path, its size and the oracle table.
Card 12's `IsNoServer` form (F3) and two wording issues on cards 7 and 3 complete the set.
