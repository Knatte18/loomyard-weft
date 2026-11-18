---
round: 1
ledger:
  - key: d8-agentconfig-premise-false
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: live-gate-names-tier-not-key
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: sandbox-reaper-outside-runproc
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: copies-cannot-get-slots-in-live-gate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: step1-tier2-run-not-serial
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: model-rotation-makes-converged-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: red-live-gate-deferred-cause-no-outcome
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stray-report-writers-archive-order
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d10-loosened-order-clause-no-reach
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: treadle-deletion-needs-parent
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: d5-reap-bounds-miss-cases
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: lightweight-budget-reader-ignores-hardener-yaml
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: harden-fix-verify-source-ambiguous
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: code-facts-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised 14 findings, three of them BLOCKING, and no earlier ledger exists.
The BLOCKING three are a false `agentconfig` premise in D8, a live gate that names a tier instead of a gate key in D4, and a sandbox reaper in D5 that ends processes outside `runproc`.
The MEDIUM design findings concern the `--copies` slot mechanics, the serial claim, model rotation reach, the live gate's exhaustion outcome and D10's loosened order clause.
The treadle deletion question is a `decision` finding for the run's parent.
