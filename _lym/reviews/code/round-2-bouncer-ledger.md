---
round: 2
ledger:
  - key: contained-tests-kept-without-fold-or-keep-reason
    rounds: [1]
    status: resolved
  - key: loom-plan-keys-refused-without-tapestry-pointer
    rounds: [1]
    status: resolved
  - key: compatibility-readers-left-behind
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: stale-anchor-refusal-remedy-broken
    rounds: [1]
    status: resolved
  - key: staged-deletion-fix-untested
    rounds: [1]
    status: resolved
  - key: sibling-wiring-walk-unasserted
    rounds: [1]
    status: resolved
  - key: rollback-marker-assertion-removed
    rounds: [1]
    status: resolved
  - key: docs-and-comments-describe-deleted-migrations
    rounds: [1]
    status: resolved
  - key: merge-danger-two-way-over-board-format
    rounds: [1]
    status: resolved
  - key: batcher-retired-key-way-forward-for-operator-profiles
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: mutation-survivors-on-changed-lines
    rounds: [1, 2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 2 maps F3, F10 and F11 onto the three keys still open; the other eight round-1 keys are resolved by the fixer's work and the review's prior answers.
F3 is the only MEDIUM; its fix text (delete the stored legacy-scope readers) is concrete, so it is marked `mechanical`.
F10 (the default-layer way forward names a verb that keeps the key) and F11 (no row seeds a prime copy while a template base exists) are LOW with fix text that names the exact edit.
