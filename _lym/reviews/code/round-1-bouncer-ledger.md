---
round: 1
ledger:
  - key: contained-tests-kept-without-fold-or-keep-reason
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: loom-plan-keys-refused-without-tapestry-pointer
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: compatibility-readers-left-behind
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: stale-anchor-refusal-remedy-broken
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: staged-deletion-fix-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: sibling-wiring-walk-unasserted
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: rollback-marker-assertion-removed
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: docs-and-comments-describe-deleted-migrations
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: merge-danger-two-way-over-board-format
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: batcher-retired-key-way-forward-for-operator-profiles
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: mutation-survivors-on-changed-lines
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
The review maps F1 to F11 one to one onto the keys above, in order.
F1 blocks on test economy; F2 to F6 are the gating design findings; F7 to F9 are consistency findings at MEDIUM; F10 and F11 are LOW.
Only F3 is marked `design-wrong`, because it chooses between deleting the remaining compatibility readers and listing them as kept.
