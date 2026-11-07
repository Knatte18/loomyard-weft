---
round: 2
ledger:
  - key: terminal-findings-drop-records-seal-faults
    rounds: [1]
    status: resolved
  - key: diff-command-marked-case-ruling-untested
    rounds: [1]
    status: resolved
  - key: amended-metadata-marks-check-untested
    rounds: [1]
    status: resolved
  - key: terminal-stop-two-way-forwards
    rounds: [1]
    status: resolved
  - key: backup-baseline-half-moved-meta
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: gate-helper-branches-untested
    rounds: [1]
    status: resolved
---

Round 1 raised six findings, one of them gating.
Round 2 re-raised only the backup edge case, now a LOW finding on the metadata rename failure path.
The other five keys were not raised again, and round 2's rejected focus items confirm the Terminal findings fix.
