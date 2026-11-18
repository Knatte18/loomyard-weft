---
round: 1
ledger:
  - key: memory-max-literal-and-constant
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: gatecli-cap-test-unbounded-hog
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: summary-other-gate-work-unproven
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: holder-end-not-pinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: holder-shares-process-group
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unaligned-cap-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: holder-never-child-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, none carried from earlier rounds.
F1 (BLOCKING) and F4 (design, MEDIUM) gate the loop.
F2, F3, F5, F6 and F7 map one to one to the keys above; each fix text in the review is local and fixes its own result.
