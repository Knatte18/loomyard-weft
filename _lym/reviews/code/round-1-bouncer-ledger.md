---
round: 1
ledger:
  - key: steer-names-awk-cat-sed-n
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: edit-directive-no-harness-override
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: looped-sed-i-spellings-pass
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: scratch-capture-source-extensions-refused
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: untested-shell-edit-pattern-branches
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: no-hookcmd-case-for-shell-edit
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised six findings, all open.
The two gating ones are the steer wording that still names awk, cat and sed -n, and the edit directive that lacks the harness override statement.
The four LOW findings cover looped `sed -i` spellings, scratch captures into source extensions, untested pattern branches and a missing hook-command case.
