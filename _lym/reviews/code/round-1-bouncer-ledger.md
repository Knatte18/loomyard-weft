---
round: 1
ledger:
    - key: printed-wait-includes-in-slot-work
      rounds:
        - 1
      status: resolved
      class: design
      severity: BLOCKING
      fix: design-wrong
    - key: rerun-line-hides-slot-wait
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: design-wrong
    - key: summary-patterns-require-wait-field
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: printed-wait-untested-on-real-paths
      rounds:
        - 1
      status: open
      class: design
      severity: MEDIUM
      fix: mechanical
    - key: startedlines-fixture-old-line-form
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
    - key: summary-line-test-repeats-reader-coverage
      rounds:
        - 1
      status: open
      class: consistency
      severity: LOW
      fix: mechanical
---
Round 1 is the first round, so every key is new.
The open `design` keys concern what the printed wait measures, how the summary line is read across binaries,
and the missing test of the wait on real tier runs.
The two `consistency` keys are test upkeep in `internal/gateslot`.
