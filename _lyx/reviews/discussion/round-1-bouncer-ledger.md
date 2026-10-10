---
round: 1
ledger:
  - key: d1-deadline-bound-outcome-done
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d6-environment-set-misses-unix-tag
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d3-contract-file-bound-false-premise
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d3-reset-own-path-set-unbounded
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-reprompt-text-cannot-carry-findings-path
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: scope-item-5-probe-history
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d6-behaviors-without-acceptance-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised seven findings, F1 to F7, in the order of the ledger keys above.
Four are `design` and gate: the D1 deadline outcome, the D6 `unix` tag gap, the D3 contract-file premise and the D3 reset own-path bound.
Three are LOW `consistency` findings on D2, Scope item 5 and the D6 acceptance rows.
