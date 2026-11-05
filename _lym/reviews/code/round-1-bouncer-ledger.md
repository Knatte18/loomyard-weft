---
round: 1
ledger:
  - key: serial-subtests-no-shared-fixture-comment
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: reset-subpath-refused-after-hub-removal
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: reset-binding-and-old-order-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: test-comments-name-wrong-nonhub-refuser
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: shared-block-comment-wrong-check
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round; no earlier ledger exists.
The BLOCKING finding is a missing comment naming the shared hub fixture of serial subtests.
The two MEDIUM findings are a `--subpath` refusal that still follows the reset's removal (deferred to the parent) and missing `--reset` tests for the binding mismatch and the old-order guard.
The LOW and NIT findings are stale or inaccurate comments.
