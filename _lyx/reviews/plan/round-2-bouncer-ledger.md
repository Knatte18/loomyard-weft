---
round: 2
ledger:
  - key: round-seam-filled-late
    rounds: [1]
    status: resolved
  - key: standalone-gate-refusal-dead-end
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
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
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: darn-wire-seed-missed-by-refusal
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: machine-layer-named-in-standalone
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 2 maps F1 to `standalone-gate-refusal-dead-end`, F2 to `verify-verb-tests-left-on-plan-section`, F3 to `darn-wire-seed-missed-by-refusal` and F4 to `machine-layer-named-in-standalone`.
The round 1 keys that round 2 did not raise again are resolved.
The one live design problem is the standalone worktree write path of `lyx config gate`: the fabric sync it always runs fails outside a hub, and the missing `_lyx` directory refuses first.
The rest are a scope gap in card 12's test edits and two wording or seed-list gaps.
