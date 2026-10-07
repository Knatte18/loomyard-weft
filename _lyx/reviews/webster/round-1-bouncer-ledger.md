---
round: 1
ledger:
  - key: fixed-column-comment-breaks
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: serial-tests-without-global-state-note
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: lint-directive-exempts-whole-block
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: lint-missing-yet-so-conjunctions
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: derive-error-always-writer-finding
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: verify-gate-refusal-row-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: cmd-lyx-budget-not-pinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: nil-index-error-untested
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: per-helper-test-covered-by-guard
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
---

Round 1 raised nine findings, all open.
They are F1 to F9 in the order of the keys above.
Five are BLOCKING and two are MEDIUM `design`.
No key has recurred, since this is the first round.
