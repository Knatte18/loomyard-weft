---
round: 1
ledger:
  - key: journal-scratch-dir-never-created
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: restore-card-name-rule-stricter-than-format
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: prose-mention-plural-list-not-reported
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: pattern-sole-parsers-write-path-list-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card4-restore-renumber-bound-misstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: archive-untouched-criterion-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card4-tests-duplicate-cards-2-3
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: non-integer-n-read-as-run-id
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: lease-test-scheduling-dependent
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: precheck-readable-journal-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first review; there is no earlier ledger.
Keys map to findings F1 through F10 in order.
The gating ones are the journal scratch directory (F1), restore-renumber's misstated bound (F5) and card 4's duplicated tests (F7).
