---
round: 1
ledger:
  - key: card10-tier0-spawns-git
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: adopt-only-records-checkout-refusal-unreached
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: reused-records-worktree-blocks-ff-fetch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: archive-compares-tips-before-fetch
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: sweeps-lock-after-state-read
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: batten-resume-refusal-lacks-follow-up
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: retry-widened-beyond-archive-tag-push
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: remove-warning-not-reaching-envelope
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: gone-pair-refusal-text-falsified
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: landing-push-noop-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: push-anchored-no-production-caller
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: config-lock-row-cannot-fail
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: falsified-doc-text-outside-card-symbols
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 is the first review; there is no earlier ledger.
Every key is open.

The gating keys are `card10-tier0-spawns-git` (BLOCKING) and four `design` keys at MEDIUM: the adopt-only refusal reach, the records-worktree fast-forward, the archive tip comparison order, and the sweeps' lock placement.
The review's F1 to F13 map to the keys in order.
