---
round: 1
ledger:
  - key: park-commit-and-commitrecords-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dirholdsfile-accepts-plain-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: duplicate-holds-file-helper
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: done-friction-failed-report-uncommitted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all open.
F1 (Park seam commit and `CommitRecords` closure untested) is the gating one.
F2 and F3 concern `dirHoldsFile` in `internal/shedrun/paths.go`: a dropped IsDir guard and a duplicate of loomcli's `holdsFile`.
F4 is the `finishDone` friction-failed report written after `CommitRecords`.
Each fix is local and its result is fixed by the finding text.
