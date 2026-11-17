---
round: 1
ledger:
  - key: overview-codex-nudge-and-deny-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: contained-tests-missing-keep-markers
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: localpath-windows-file-url
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-rollout-and-missing-ids-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: breach-switches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: validate-batching-nudge-accepting-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: countlinesupto-limit-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: rollout-branches-fixture-never-writes
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: lines-separate-argument-row-missing
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised F1 to F9: two consistency findings (docs, keep markers) and seven design findings, one a Windows path bug
and the rest test gaps.
All are LOW or NIT with mechanical fixes, so none gates.
