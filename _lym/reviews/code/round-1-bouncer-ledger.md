---
round: 1
ledger:
  - key: transcript-filename-literal-repeated
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: read-failure-after-paste-requeues-command
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: orch-start-waits-before-save-and-register
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: disconnect-case-narrowed-unrecorded
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
  - key: unconfirmed-paste-skips-wait
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: resume-fallback-checkpoint-reset-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: claude-checkpoint-connected-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: remote-control-error-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: precheckpoint-ordering-not-pinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: merge-danger-blast-radius-narrow
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: contained-test-needs-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: bridge-record-without-id-unpinned
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: empty-session-id-unpinned
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: new-test-files-lack-file-comment
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised 14 findings, all open.
Gating: the repeated transcript file-name literal (BLOCKING), a read failure after the paste that makes creel retype the command, and the orch start's connection wait before save and register (both MEDIUM design).
The rest are LOW and NIT test gaps and consistency items; the disconnect narrowing is deferred to the parent.
