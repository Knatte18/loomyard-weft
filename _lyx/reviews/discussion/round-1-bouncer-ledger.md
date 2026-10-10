---
round: 1
ledger:
  - key: empty-optional-directives-fail-table-validation
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: empty-scalar-breaks-advisor-count-bound
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: advisor-bounds-stencil-text-only
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: amendment-path-unscoped-built-ahead-of-need
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: advisor-reliance-two-records-no-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: d3-bound-misses-live-old-binary-writer
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: rejected-alternative-in-d5-rationale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: opening-stencil-role-keys-disagree
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

First round, no prior ledger.
Round 1 raised eight findings: the BLOCKING one is that optional directives, empty when PATTERN is inactive or Tier 2 is off, fail the seat engine's blank-value rule.
Three design MEDIUMs cover the empty-scalar advisor list, unstated advisor bounds and the unscoped amendment path.
Four consistency findings are LOW or NIT.
