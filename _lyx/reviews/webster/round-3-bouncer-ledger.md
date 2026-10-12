---
round: 3
ledger:
  - key: publish-comments-stale-merge-in
    rounds: [1]
    status: resolved
  - key: lyxtemp-concurrent-sweep-lock-probe-race
    rounds: [2]
    status: resolved
  - key: lyxtemp-remove-retry-no-pause
    rounds: [3]
    status: open
    class: design
    severity: LOW
---

Round 1's stale merge-in wording NIT stayed resolved.
Round 2's BLOCKING lock-probe race in concurrent sweeps is resolved: round 3 judges the bounded `removeRunDirectory` retry correct.
Round 3 raised one LOW design finding: the retry has no pause between attempts, so on Windows it can land inside the peer probe's hold of the recreated lock file.
