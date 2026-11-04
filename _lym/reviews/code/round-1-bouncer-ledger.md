---
round: 1
ledger:
  - key: add-hold-span-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: create-closure-hold-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: goal-ideas-unmade-no-decision
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: docs-claim-no-network-call-under-hold
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: add-hold-teardown-lock-order-unrecorded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: primelockwait-header-names-old-callers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: second-hold-after-release-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings: two MEDIUM `design` test gaps (F1, F2), one MEDIUM `decision` deferred to the parent (F3), and four LOW (F4 to F7).
F1 and F2 gate; the rest do not.
F3's fix leaves the code in place and defers to the parent's `lym loom decision add`.
No earlier ledger exists, so nothing is carried forward.
