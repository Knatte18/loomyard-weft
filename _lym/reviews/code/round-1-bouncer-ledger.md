---
round: 1
ledger:
  - key: lock-free-pass-skips-owner-checks
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: tip-read-queues-at-back
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: recipe-comments-old-lock-order
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: behind-upstream-path-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: lock-free-fixer-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: held-flip-counts-false-lost-race
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: config-change-notice-base-moved
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: new-error-paths-and-front-queue-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: locked-verify-skip-not-shown
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: finalize-and-freelock-docs-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: prepush-exemption-rationale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round.
The review finds every point of the board entry built, and its findings concern what the new lock-free pass changed around it.

The main defect is ordering: the lock-free pass merges, gates and fixes before the owner checks run.
The prepush exemption finding depends on how that is fixed.
The rest are lock-queue position, lost-race accounting, the config-change notice base, missing test coverage and stale prose.
