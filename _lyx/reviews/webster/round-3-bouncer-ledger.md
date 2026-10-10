---
round: 3
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
    status: open
    class: design
    severity: BLOCKING
  - key: isreadonlycommand-doc-glob-claim-too-broad
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
  - key: errrecoveryexhausted-doc-names-one-producer
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised three findings, all resolved in round 2.
Round 3 raised three new keys.
The `cmd/lyx` test budget row is below the package's top-level test count after the merge-in, which is the Publish failure.
The `IsReadOnlyCommand` doc states a quoted glob is text wherever a positional is admitted, which is false for `lyx` segments.
The `ErrRecoveryExhausted` doc names one producer of a sentinel that has several.
