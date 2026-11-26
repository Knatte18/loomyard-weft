---
round: 1
ledger:
    - key: records-side-clean-judged-against-moving-head
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: restarted-gate-loses-prefix-head
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: fixed-head-tolerates-unvetted-commits
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: master-stencil-still-limits-to-two-contract-files
      rounds:
        - 1
      status: open
      class: consistency
      severity: MEDIUM
      fix: mechanical
    - key: master-note-path-save-and-read-untested
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: fixed-head-probe-error-reads-as-refusal
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: fixed-head-refusing-branches-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: gate-state-branches-unpinned
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: reset-fixed-head-clear-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: link-resolved-note-spelling-untested
      rounds:
        - 1
      status: open
      class: design
      severity: LOW
      fix: mechanical
    - key: orch-role-names-only-two-contract-files
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: stencil-points-at-wrong-friction-section
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: accept-audit-help-unrequested-rewording
      rounds:
        - 1
      status: open
      class: scope
      severity: LOW
      fix: mechanical
    - key: records-side-ignored-and-reports-rows-missing
      rounds:
        - 1
      status: open
      class: design
      severity: NIT
      fix: mechanical
---
Round 1 is the first round, and every finding is open.

The open design problems are in two places.
The records-side acceptance check compares against a records HEAD that batch-boundary syncs move.
The verify-gate fixed-head tolerance loses its base on a restart and covers a wider range than the gate vetted.
The rest are missing test rows, prose that still limits Master to its two contract files, and one help rewording.
