---
round: 2
ledger:
  - key: weaver-config-keys-named-bare
    rounds: [1]
    status: resolved
  - key: verify-span-intervals-untested
    rounds: [1]
    status: resolved
  - key: unspawned-fixer-records-rework-span
    rounds: [1]
    status: resolved
  - key: attached-fixer-no-span-unpinned
    rounds: [1]
    status: resolved
  - key: attempt-timeout-comments-omit-attach
    rounds: [1]
    status: resolved
---

Round 1 raised every key above: bare config key names, span tests that compared kinds but not intervals,
a rework span for an unspawned fixer, timeout comments that omitted the attach case,
and a missing assertion on an attached fixer.
Round 2 found all of them resolved and raised no new finding.
