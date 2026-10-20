---
round: 1
ledger:
  - key: darn-tree-scope-reader-drops-editor-way-forward
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: editor-no-tree-way-forward-chosen-by-sentinel
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: plan-verify-misses-contract-packages
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: run-start-way-forward-says-resume
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: set-way-forward-cannot-write-dotted-tree
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: classifier-guards-impossible-states
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: darn-recipe-gate-comment-stale
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: one-lyx-build-per-guard-package
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: merge-in-skip-step-duplicates-verifytree-tests
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings, none carried from an earlier round.
The blocking defect is that the per-call tree-scope reader returns config load errors without the editor way forward.
The other gating item is the plan's `verify:` block, which misses two contract packages.
The rest are wording, coverage and test-economy fixes, plus a guard-step batching change in the derived gate command.
