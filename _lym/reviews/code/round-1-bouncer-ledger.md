---
round: 1
ledger:
  - key: line-group-killed-outside-runproc
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: delivered-mark-after-failed-copy
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-lease-cancels-healthy-run
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: finished-run-marked-abandoned-race
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: process-file-visible-before-content
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: run-directories-never-pruned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: verb-blind-text-unchecked-card-verify
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: zero-cap-inherited-slot-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: overview-omits-keep-warm-keys
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

First round, no prior ledger.
F1 is the only MEDIUM and is a parent decision on `PATTERN-run-process-owner` versus D6's `proc.KillTree`.
The rest are LOW local corrections.
