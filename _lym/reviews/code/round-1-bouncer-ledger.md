---
round: 1
ledger:
  - key: reconcile-unparseable-profiles-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reconcile-envelope-lacking-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dry-run-migrated-lacking-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: migrateconfig-non-mapping-root-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: walk-lacking-warn-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: differing-former-block-discarded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1 is the first review; there is no earlier ledger.
Two MEDIUM design findings gate: the reconcile path for a profiles.yaml that fails to parse, and the envelope's lacking key, both lack tests.
Three LOW design findings are further test gaps: dry-run assertions, the non-mapping root refusal, and the walk's Warn.
One LOW decision finding asks the author to settle whether a differing former block beside a current one is dropped or kept.
