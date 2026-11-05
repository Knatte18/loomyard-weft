---
round: 2
ledger:
  - key: serial-subtests-no-shared-fixture-comment
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: reset-subpath-refused-after-hub-removal
    rounds: [1]
    status: resolved
  - key: reset-binding-and-old-order-untested
    rounds: [1]
    status: resolved
  - key: test-comments-name-wrong-nonhub-refuser
    rounds: [1, 2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: shared-block-comment-wrong-check
    rounds: [1]
    status: resolved
  - key: summary-contradicts-diff-on-subpath-check
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: recorded-anchor-on-weft-branches-untested
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
---

Round 1's BLOCKING comment finding was fixed but left two serial subtests uncommented (F3, LOW).
The `--subpath` ordering fix landed through `checkSubpathBeforeReset`, and the binding and old-order tests exist, so those keys are resolved.
A stale `resetHub` attribution remains in the neutering log (F4, NIT).
New in round 2: the change description contradicts the diff on the `--subpath` check (F1), and `recordedAnchorOnWeft` has untested branches (F2).
No finding is `design` class.
