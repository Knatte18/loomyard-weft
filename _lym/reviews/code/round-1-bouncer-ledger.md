---
round: 1
ledger:
  - key: orphan-stop-no-recheck-before-stop
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: no-probe-assertion-vacuous
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reconcile-hook-error-path-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: restep-ambiguous-in-shedverbs-doc
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: stepstopnote-error-wording-names-one-step
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: stoporphanedstepruns-doc-names-none
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: gone-row-seeds-non-positive-pid
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, all LOW or NIT, none gating.
F1 (re-check before stop), F2 (vacuous no-probe assertion) and F3 (reconcile hook error path) are design-class LOW.
F4 to F7 are wording and test-row consistency fixes.
No earlier ledger exists.
