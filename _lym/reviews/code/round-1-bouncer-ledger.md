---
round: 1
ledger:
  - key: resume-at-reconcile-clears-steps
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: held-back-way-forward-refused-by-standing-record
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stopped-part-clearing-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: resume-reresolves-all-against-live-board
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: converted-owner-entry-loses-parent-lock-exemption
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dobby-owner-lift-adopt-pause-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: tender-pause-already-paused-adopt-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: envelope-reconcile-mutations-dropped
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: testadoptfields-needs-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: testtriesstore-needs-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 of the code review: ten findings, none carried from an earlier round.
Three are gating: F1 (`worker.go` resume at reconcile clears steps), F2 (`steps.go` held-back way forward fails `SameSubjects`), F3 (no test pins stopped-part clearing).
The rest are LOW: F4 resume re-resolves `--all`, F5 pauserecord doc silence on converted entries, F6 and F7 missing D5 acceptance tests, F8 dropped reconcile mutations, F9 and F10 keep markers for two contained tests.
