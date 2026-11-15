---
round: 1
ledger:
  - key: untracked-files-escape-worktree-check
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: batten-arbiter-not-widened
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: readedits-read-failure-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: failed-commit-read-hides-head-moved
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: combined-edit-outcomes-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: seam-commits-passthrough-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: verb-commits-key-uncovered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: testprompt-needs-keep-marker
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: fix-stencil-drops-no-mutating-git-rule
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: loomcli-doc-arbiter-section-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 was the first review, so there was no earlier ledger to carry forward.
The keys map to the review's findings F1 to F10 in order.
F1 is the one BLOCKING finding: the worktree check ignores untracked files, and its fix extends the `WorktreeSeam` interface.
F2 is a decision for the run's parent and the fixer leaves the code in place.
F3 is the only other gating finding.
The rest are local test, doc and stencil corrections.
