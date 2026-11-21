---
round: 1
ledger:
  - key: leftover-refusal-short-branch-delete
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: records-archive-tag-manual-push-no-destination
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: seed-upstream-config-failure-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: push-retry-namespaced-tag-row-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gitrepo-doc-stale-mandatory-bullet
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised five findings (F1 to F5), none BLOCKING.
F1 and F2 are MEDIUM consistency findings about manual way-forward commands that cannot reach the namespaced refs.
F3 and F4 are LOW design findings, and F5 is a NIT on a stale doc bullet.
Nothing gates, so the loop converges.
