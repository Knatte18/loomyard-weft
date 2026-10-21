---
round: 4
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
    rounds: [3, 4]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: verify-fixture-commit-exclude-missing
    rounds: [3]
    status: resolved
  - key: tier-lists-no-seam-to-shed-and-landing
    rounds: [3]
    status: resolved
  - key: drop-warning-names-wrong-layer
    rounds: [3]
    status: resolved
  - key: absent-gate-yaml-refused-at-wiring
    rounds: [4]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: decodeconfig-resign-arrow-missing
    rounds: [4]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 4 maps F1 to `shared-fixtures-missed-by-new-behavior`, F2 to `absent-gate-yaml-refused-at-wiring` and F3 to `decodeconfig-resign-arrow-missing`.
F1 is the round 3 method gap on new instances: reflection-driven seam inventories, hand-built fixtures that bypass a production constructor, and tests bound to a retired input, on cards 10, 12, 13, 15 and 19.
F2 is a wiring contract conflict: cards 16 and 18 refuse an absent `gate.yaml` where card 17 and `wireHub` treat it as tolerable.
F3 is a missing re-sign arrow on `decodeConfig` and its unlisted caller `TemplateConfig`.
The round 3 keys on commit and exclude, tier-list seams and the drop-warning layer are resolved.
