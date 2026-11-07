---
round: 1
ledger:
  - key: model-default-not-rendered
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: overview-omits-effort-defaults-silently
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: replayer-discard-of-commit-overview-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-branches-without-row
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-replayer-registration-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: blank-line-lost-before-heading
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings (F1 to F6, in the order above), none BLOCKING.
F1 is a MEDIUM `decision` finding deferred to the parent; the rest are LOW.
No key recurs and none was open earlier.
