---
round: 1
ledger:
  - key: harden-bouncer-first-raised-gates-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: judge-stencil-comment-stale-decision-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: test-names-describe-removed-focus-permission
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, none gating: untested `FirstRaisedGates` wiring on the `harden_round` Bouncer,
a stale maintainer comment in the judge stencil, and test names that still describe the removed focus-finding permission.
