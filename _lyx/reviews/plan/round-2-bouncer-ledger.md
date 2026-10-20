---
round: 2
ledger:
  - key: darn-tree-scope-reader-drops-editor-way-forward
    rounds: [1]
    status: resolved
  - key: editor-no-tree-way-forward-chosen-by-sentinel
    rounds: [1]
    status: resolved
  - key: plan-verify-misses-contract-packages
    rounds: [1]
    status: resolved
  - key: run-start-way-forward-says-resume
    rounds: [1]
    status: resolved
  - key: set-way-forward-cannot-write-dotted-tree
    rounds: [1]
    status: resolved
  - key: classifier-guards-impossible-states
    rounds: [1]
    status: resolved
  - key: darn-recipe-gate-comment-stale
    rounds: [1]
    status: resolved
  - key: one-lyx-build-per-guard-package
    rounds: [1]
    status: resolved
  - key: merge-in-skip-step-duplicates-verifytree-tests
    rounds: [1]
    status: resolved
  - key: landing-sites-editor-way-forward-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: darn-tree-scope-drops-verify-way-forward
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: publish-seam-error-compared-to-nonfailing-read
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: verify-block-omits-websterengine
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised nine findings; round 2 does not raise any of them again, so all are carried as resolved.
Round 2 raised four new findings: two MEDIUM scope findings about the way forward at Merge-In, Finalize and Publish (F1 test coverage, F2 the seam dropping `VerifyCommand`'s clause), a LOW wording fix in Card 8 (F3), and a LOW verify-block addition of `./internal/websterengine` (F4).
None is BLOCKING or a design-class finding at MEDIUM or worse.
