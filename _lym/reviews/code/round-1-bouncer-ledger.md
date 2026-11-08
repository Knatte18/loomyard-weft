---
round: 1
ledger:
  - key: presence-notice-skip-guards-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: orch-marked-woken-when-forward-posted-nothing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: retried-forward-loses-presence-skip
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: stale-turnhookcommand-doc
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: backoff-reset-and-boundary-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
F1 (untested presence-notice skip guards) is the one gating finding.
F2 and F3 are two failure paths where the skip is wrongly applied or lost; F3 needs a new pending set, so its fix changes the design.
F4 is a stale field doc, and F5 is a missing backoff test.
