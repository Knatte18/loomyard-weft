---
round: 1
ledger:
    - key: config-keys-bare-and-literal
      rounds:
        - 1
      status: resolved
      class: consistency
      severity: BLOCKING
      fix: mechanical
    - key: tests-hand-write-module-config
      rounds:
        - 1
      status: resolved
      class: consistency
      severity: BLOCKING
      fix: mechanical
    - key: untested-holder-in-task-worktree-itself
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: untested-blocked-reason-selection
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: untested-second-stop-episode
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: untested-failed-record-holds-clear
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: untested-gate-hold-hub-wiring
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: child-read-newer-than-derived-state
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: batten-reason-absolute-without-child
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: windows-drops-holder-tree
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: cleanup-help-omits-watch-holds
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: untested-time-or-reason-only-change
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: untested-gate-holds-survive-watch-rewrite
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: untested-ended-hold-pruning
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: untested-reread-on-batten-status-write
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: untested-largest-timeout-of-mixed-hold
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: untested-watch-log-line
      rounds:
        - 1
      status: open
      class: design
      severity: NIT
      fix: mechanical
    - key: untested-failing-status-exists
      rounds:
        - 1
      status: open
      class: design
      severity: NIT
      fix: mechanical
    - key: non-finite-gate-hold-factor-accepted
      rounds:
        - 1
      status: open
      class: design
      severity: NIT
      fix: mechanical
    - key: holder-marker-doc-overstates-reach
      rounds:
        - 1
      status: open
      class: consistency
      severity: NIT
      fix: mechanical
---
Round 1 is the first round.
The blocking items are two repository-rule violations: how new text names config keys, and how new tests write config files.
The gating design findings at MEDIUM are all missing test coverage of the run watch, the done-run clear and the gate-hold notice.
The lower findings are small behaviour corrections, further coverage rows and two stale descriptions.
