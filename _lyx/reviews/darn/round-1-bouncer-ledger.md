---
round: 1
ledger:
  - key: empty-scope-no-line-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: trimpath-line-last-field-wins
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: trimpath-line-bool-spellings
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: guard-step-line-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four `design` findings.
The MEDIUM one is a missing test for the empty-scope guard.
The three LOW ones are two `trimpathLine` parsing gaps (repeated flag, other bool spellings) and an unpinned guard-step line.
Every fix is local and fully specified by the finding text.
