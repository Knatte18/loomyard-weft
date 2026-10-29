---
round: 1
ledger:
  - key: card6-misses-superseded-hook-wait-subtest
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: orch-stencil-keeps-bell-first-wake
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: d3-bounds-and-delivered-once-exceptions-undocumented
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: pause-read-misses-superseding-waiter
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: ring-path-intent-lacks-attack-surface-bound
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card3-release-widened-to-read-verb
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: design-wrong
  - key: release-logs-second-line-for-empty-claim
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card7-paused-or-held-row-leaves-paused-claim-end-open
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: smoke-five-line-check-counts-tokens-in-whole-pane
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card1-testhookcontext-row-count
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so no earlier ledger exists.
The review raised ten findings, none BLOCKING and none a design-class MEDIUM, so the loop converged.
The keys above map to F1 through F10 in order.
