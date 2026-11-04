---
round: 1
ledger:
  - key: capture-failure-subtest-timing-and-untested-exit
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: capture-verb-exits-zero-before-baseline-check
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: command-report-reads-whole-log-and-drops-last-line
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reconcile-notifier-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: null-dir-flagged-as-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: failure-text-and-hook-branches-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reinterpretation-walk-delivery-and-rewrite-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reinterpreted-marker-doc-names-sibling-symbol
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised eight findings, F1 to F8, mapped in order to the keys above.
F1 (BLOCKING), F3 and F4 are gating.
F2 is a `decision` finding for the run's parent to settle with `lym loom decision add`; its code stays in place.
F5 to F8 are low-severity test gaps and a doc wording.
