---
round: 2
ledger:
  - key: lock-free-pass-skips-owner-checks
    rounds: [1]
    status: resolved
  - key: tip-read-queues-at-back
    rounds: [1]
    status: resolved
  - key: recipe-comments-old-lock-order
    rounds: [1]
    status: resolved
  - key: behind-upstream-path-untested
    rounds: [1]
    status: resolved
  - key: lock-free-fixer-untested
    rounds: [1]
    status: resolved
  - key: held-flip-counts-false-lost-race
    rounds: [1]
    status: resolved
  - key: config-change-notice-base-moved
    rounds: [1]
    status: resolved
  - key: new-error-paths-and-front-queue-untested
    rounds: [1]
    status: resolved
  - key: locked-verify-skip-not-shown
    rounds: [1]
    status: resolved
  - key: finalize-and-freelock-docs-stale
    rounds: [1]
    status: resolved
  - key: prepush-exemption-rationale
    rounds: [1]
    status: resolved
---

Round 1 found the board entry built, with one BLOCKING ordering defect:
the lock-free pass merged, gated and fixed before the owner checks ran.
Its other findings concerned lock-queue position, lost-race accounting, the config-change notice base,
missing test coverage and stale prose.

Round 2 marks every one of them resolved and raises nothing new.
