---
round: 1
ledger:
  - key: reopen-merged-pr-keeps-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: removeready-unready-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: readreopenrequest-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: pr-reviewer-stop-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reopen-request-removal-failure-ignored
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unready-commit-empty-pathspec
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: dobbygeom-test-omits-reopen-request
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: beamdescribedeps-stale-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised eight findings: four MEDIUM design (F1 to F4), two LOW design (F5, F6) and two LOW consistency (F7, F8).
Keys map in order: F1 `reopen-merged-pr-keeps-record`, F2 `removeready-unready-untested`, F3 `readreopenrequest-untested`, F4 `pr-reviewer-stop-unpinned`, F5 `reopen-request-removal-failure-ignored`, F6 `unready-commit-empty-pathspec`, F7 `dobbygeom-test-omits-reopen-request`, F8 `beamdescribedeps-stale-comment`.
No earlier ledger exists.
