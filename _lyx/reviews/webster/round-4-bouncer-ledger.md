---
round: 4
ledger:
  - key: git-config-flag-row-misses-flag-check
    rounds: [1]
    status: resolved
  - key: pathless-violation-classes-hand-copy
    rounds: [1]
    status: resolved
  - key: redundant-glob-operand-readonly-lyx-segment
    rounds: [1]
    status: resolved
  - key: test-budget-row-stale-after-merge-in
    rounds: [3]
    status: resolved
  - key: isreadonlycommand-doc-glob-claim-too-broad
    rounds: [3]
    status: resolved
  - key: errrecoveryexhausted-doc-names-one-producer
    rounds: [3]
    status: resolved
---

Round 1 raised three findings, all resolved in round 2.
Round 3 raised three keys: the stale `cmd/lyx` budget row (the Publish failure), the over-broad glob claim in the `IsReadOnlyCommand` doc, and the `ErrRecoveryExhausted` doc naming one producer.
Round 4 found no defect: the budget row covers the counted tests, and both docs match the code.
