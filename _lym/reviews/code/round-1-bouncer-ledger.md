---
round: 1
ledger:
  - key: failed-stop-records-batch-kind-for-needs-parent
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reset-foreign-path-check-before-strand-probe
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rename-through-weaver-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: true-last-batch-rendering-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: state-write-failure-after-outcome-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stop-reason-truncation-cuts-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reset-does-not-hold-run-lock
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: fault-outcome-climbs-a-rung
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-doc-says-begin-clears-stop-record
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-cwd-resolution-omits-weaverengine
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reentry-card-hash-refresh-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reset-cli-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised twelve findings, none recurring.
Five are design-class MEDIUM and gate: the failed-stop kind for a needs-parent attempt (F1), the reset order of foreign-path check and strand probe (F2), and missing test rows for rename, true-last-batch and state-write failure (F3, F4, F5).
The remaining seven are LOW: stop-reason truncation (F6), reset run lock (F7), fault rung climb (F8), stale doc comments (F9), the pattern doc list (F10), re-entry hash refresh test (F11) and reset wiring test (F12).
Every fix is a local correction its finding text spells out.
