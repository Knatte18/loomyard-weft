---
round: 1
ledger:
  - key: missing-sessions-dir-reads-unreadable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: record-before-relaunch-order-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-hold-release-leaves-strand-held
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: turn-ends-counted-from-attach-cursor
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-home-accepts-tilde-config-dir
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reed-resume-help-text-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: unrelated-testcover-change
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: contained-tests-lack-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: orphan-and-step-stop-of-relaunching-run-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: relaunch-and-orch-restart-failure-branches-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: repeated-launch-of-resumed-session-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: pick-names-read-under-lock-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: known-directory-sources-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 is the first review, so no earlier ledger exists.
F1 (`missing-sessions-dir-reads-unreadable`) and F2 (`record-before-relaunch-order-untested`) are gating design findings at MEDIUM.
The remaining findings are LOW: three more design findings, two consistency findings and six scope findings, mostly missing test rows.
Every fix text names its concrete edit, so each is marked mechanical.
