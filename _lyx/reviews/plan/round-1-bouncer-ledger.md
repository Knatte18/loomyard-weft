---
round: 1
ledger:
  - key: round-seam-filled-late
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: standalone-gate-refusal-dead-end
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: verifytree-unslotted-too-early
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: base-scope-no-guard-step
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: two-module-scope-row-unreachable
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: gateslot-testmain-limits-missed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: configcli-darn-fixture-missed
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: timeout-way-forward-bad-key
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card-verify-wrong-audience
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: pretier-ceiling-no-context
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: env-gateslots-orphaned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: running-tests-guide-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: drain-bound-overstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: duplicate-scenario-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: cardgatecommand-redundant-rows
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: verifytiersvalue-stale-name
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 maps F1 to F16 in order onto the keys above.
The BLOCKING design findings concern sequencing (the round seam filled by a later card) and an unwritable standalone `gate.yaml`.
The remaining design finding is the premature removal of verifytree's slot and timeout.
The rest are scope gaps, test-economy trims and wording fixes.
