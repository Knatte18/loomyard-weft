---
round: 1
ledger:
  - key: walk-misses-strict-load-routes
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: board-rewrite-assumes-restore-and-commit
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: walk-runs-on-verbs-loading-no-config
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: hubreconcile-doc-start-verb-wording-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: config-layers-background-keeps-unknown-key-rule
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: classification-test-called-unchanged
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: move-parse-failure-branch-unreached
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: seed-child-way-forward-checked-on-authored-text
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreadable-board-row-reaches-missing-config
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: profiles-write-rule-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 opened every key above.
The two blocking keys are both places where the plan's description of existing code does not hold:
which routes load the loom or tapestry config strictly (card 9), and what the board write path does on failure (card 8).
The rest are local corrections to card text, planned test rows and stale documentation.
