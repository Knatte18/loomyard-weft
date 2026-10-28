---
round: 1
ledger:
  - key: loomcli-program-name-resolver-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 only: no test pins a failing LOOM program-name resolver, so `noticeFits` passing on a geometry error and `decision add` not aborting are unguarded.
It is LOW and non-gating; the fix is a local test row whose content the finding text already fixes.
