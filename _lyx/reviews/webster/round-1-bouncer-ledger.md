---
round: 1
ledger:
  - key: computed-rm-hook-test-unpinned-behaviors
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised F1 only: `TestComputedRmDenyHook_DeniesRmOnComputedPathsOnly` pins no rm reached through a separator, pipe or xargs, and no special-parameter pass-through.
There was no prior ledger.
