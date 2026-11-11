---
round: 1
ledger:
  - key: reserved-launchers-name-kept
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: mechanical
  - key: refusal-envelope-shape-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: launchers-symlink-sweep-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: dangling-refuseuncontainedpath-comments
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: cmd-launcher-names-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: hubrel-in-unrelated-file
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised six findings, none recurring.
F2 and F3 are gating design findings at MEDIUM, so the loop continues.
F1 is deferred to the run's parent as an unrecorded correction.
F4, F5 and F6 are local corrections the findings already spell out.
