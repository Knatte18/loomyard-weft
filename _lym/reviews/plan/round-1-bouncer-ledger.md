---
round: 1
ledger:
  - key: fabricengine-snapshot-tests-import-cycle
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: protected-refusal-skips-retry-and-judgment
    rounds: [1]
    status: open
    class: decision
    severity: BLOCKING
    fix: design-wrong
  - key: exhausted-push-retried-as-transport
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: unset-run-records-seam-surfaces-mid-push
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: seam-test-search-misses-indirect-readers
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: moved-board-tests-lose-fixtures
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: codetip-test-keeps-shed-imports
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: run-lock-adapter-check-premature-and-duplicate
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: seam-installer-test-cites-unreachable-helper
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: uncontained-write-allowlist-reason-false
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: recorded-run-reads-decode-shed-record-in-fabric
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: outcome-class-doc-listed-under-extends
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: remote-only-needs-tips-before-no-origin-answer
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: ended-teardown-rule-run-and-lock-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.
The blocking keys are an import cycle in two planned test files and two places where card 7's outcome mapping changes what a push does against today's code.
The other themes are the required-seam installer's reach (its refusal point, its test search and its test),
and test moves whose fixtures or imports the cards do not carry.
