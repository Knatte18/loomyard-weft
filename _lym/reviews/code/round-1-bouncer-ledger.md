---
round: 1
ledger:
  - key: owned-branch-literals-in-test-expectations
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: descendant-origin-board-called-another-clones
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: refusal-branches-without-test-rows
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unwithdrawn-board-failure-names-looping-rerun
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: restore-claimed-before-it-runs
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: refusal-read-errors-lose-context
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unborn-board-message-reads-no-commits
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings: F1 (BLOCKING, consistency), F2, F3 and F4 (MEDIUM, design), F5 and F6 (LOW), F7 (NIT).
There is no earlier ledger, so nothing carries forward.
F2 and F4 change how a refusal is classified and which way forward it names, so their fixes need a fresh review.
