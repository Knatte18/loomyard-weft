---
round: 1
ledger:
  - key: terminal-findings-drop-records-seal-faults
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: diff-command-marked-case-ruling-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: amended-metadata-marks-check-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: terminal-stop-two-way-forwards
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: backup-baseline-half-moved-meta
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-helper-branches-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings.
The one gating finding is the Terminal findings dropping records and seal faults.
The rest are test gaps, a doubled way forward and a backup edge case, all with local fixes.
