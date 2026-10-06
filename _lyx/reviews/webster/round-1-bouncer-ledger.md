---
round: 1
ledger:
  - key: fold-tables-missing-t-parallel
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: hard-wrapped-new-comments
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: stale-test-name-in-batten-bookend-background
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: helpers-left-without-caller
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
---

Round 1 raised four findings, all open.
Fold tables written by the prune lack `t.Parallel` or a comment naming the global state that forbids it.
New comments are hard-wrapped mid-sentence.
`pattern/PATTERN-batten-bookend.md` names a top-level test the scenario merge turned into a step.
Three test helpers lost their callers in the prune.
