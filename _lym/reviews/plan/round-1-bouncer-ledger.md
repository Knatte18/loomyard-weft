---
round: 1
ledger:
  - key: lease-and-pointer-outside-state
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: card11-breaks-help-subtest
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card10-tests-unexported-helpers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: wait-reason-texts-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: uncapped-inprocess-wait-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runner-removed-verify-line
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: none-row-asserts-absent-log
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: await-go-docs-false
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: extends-on-rewritten-doc-sentences
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings: F1 to F9, in the order of the keys above.
F1 is BLOCKING and F3 is the only MEDIUM `design` finding, so the loop continues.
Every fix is a local text correction the finding already states.
