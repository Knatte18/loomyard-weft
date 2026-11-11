---
round: 1
ledger:
  - key: card4-unexported-helper-tests-repeat-name-set
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card1-competing-write-subtest-redundant
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: card1-takeover-docs-false-with-replace-set
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card4-unpushed-read-error-no-outcome
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: card4-failed-hub-way-forward-untestable-condition
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card4-fakehubs-seam-count
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review, so there is no prior ledger.
Two BLOCKING test-economy findings (card 4's helper tests, card 1's competing-write subtest) drive `CONTINUE`.
The remaining four are a MEDIUM doc-consistency gap on card 1, a LOW unspecified error outcome and a LOW untestable way-forward condition on card 4, and a NIT seam count.
