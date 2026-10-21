---
round: 3
ledger:
  - key: round-seam-filled-late
    rounds: [1]
    status: resolved
  - key: standalone-gate-refusal-dead-end
    rounds: [1, 2]
    status: resolved
  - key: verifytree-unslotted-too-early
    rounds: [1]
    status: resolved
  - key: base-scope-no-guard-step
    rounds: [1]
    status: resolved
  - key: two-module-scope-row-unreachable
    rounds: [1]
    status: resolved
  - key: gateslot-testmain-limits-missed
    rounds: [1]
    status: resolved
  - key: configcli-darn-fixture-missed
    rounds: [1]
    status: resolved
  - key: timeout-way-forward-bad-key
    rounds: [1]
    status: resolved
  - key: card-verify-wrong-audience
    rounds: [1]
    status: resolved
  - key: pretier-ceiling-no-context
    rounds: [1]
    status: resolved
  - key: env-gateslots-orphaned
    rounds: [1]
    status: resolved
  - key: running-tests-guide-stale
    rounds: [1]
    status: resolved
  - key: drain-bound-overstated
    rounds: [1]
    status: resolved
  - key: duplicate-scenario-rows
    rounds: [1]
    status: resolved
  - key: cardgatecommand-redundant-rows
    rounds: [1]
    status: resolved
  - key: verifytiersvalue-stale-name
    rounds: [1]
    status: resolved
  - key: verify-verb-tests-left-on-plan-section
    rounds: [2]
    status: resolved
  - key: darn-wire-seed-missed-by-refusal
    rounds: [2]
    status: resolved
  - key: machine-layer-named-in-standalone
    rounds: [2]
    status: resolved
  - key: shared-fixtures-missed-by-new-behavior
    rounds: [3]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: verify-fixture-commit-exclude-missing
    rounds: [3]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: tier-lists-no-seam-to-shed-and-landing
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: drop-warning-names-wrong-layer
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 3 maps F1 to `shared-fixtures-missed-by-new-behavior`, F2 to `verify-fixture-commit-exclude-missing`, F3 to `tier-lists-no-seam-to-shed-and-landing` and F4 to `drop-warning-names-wrong-layer`.
The round 2 keys are resolved: the standalone config write path traces ok, card 12 lists the verify-verb tests, and the darn seeds are reworded.
F1 is a method gap: shared fixtures and sibling tests reached by a changed reader or skip rule are unlisted in three cards.
F3 is the live design problem: the old-record and fallback tier mapping reads gate lists that no seam hands to `loomshed` or `landingshed`.
