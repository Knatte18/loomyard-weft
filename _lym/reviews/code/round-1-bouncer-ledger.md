---
round: 1
ledger:
  - key: fronttask-startaction-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: fronttask-file-record-write-failure-reported-not-filed
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: darnwrite-condition-stop-empty-reason
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: green-resume-reads-other-subjects-children
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hubsettledeps-owner-read-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-omits-front-task
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all first-time keys.
The one gating finding is the missing test for `startAction` in `internal/fronttask`.
The rest are LOW and local.
