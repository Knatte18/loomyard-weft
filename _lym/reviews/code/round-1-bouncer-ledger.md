---
round: 1
ledger:
  - key: two-root-helpers-no-split-root-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: verbs-fixture-plan-has-no-root
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised two findings, both test-coverage gaps in the subpath-anchor change.
F1 maps to `two-root-helpers-no-split-root-test`: the engine helpers taking `PaneCwd` and `WorktreeRoot` are only exercised with equal roots.
F2 maps to `verbs-fixture-plan-has-no-root`: the webstercli verbs fixture plan has no `root:`, so dispatch, done-check and size paths can resolve against the anchor unnoticed.
Neither key has a prior round.
