---
round: 1
ledger:
  - key: weaver-config-keys-named-bare
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: verify-span-intervals-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unspawned-fixer-records-rework-span
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: attempt-timeout-comments-omit-attach
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: attached-fixer-no-span-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised every key above, and there is no earlier ledger.
The gating items are the bare config key names in the weaver package doc and the span tests that compare kinds but not intervals.
The rest are LOW: a rework span for a fixer that never spawned, timeout comments that omit the attach case,
and a missing resume assertion on an attached fixer.
