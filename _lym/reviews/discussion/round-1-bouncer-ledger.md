---
round: 1
ledger:
  - key: d2-generation-premises-false
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d3-clear-reuses-ask-key
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-cap-formula-unstated
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-no-parent-unreachable
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-answer-names-no-ask
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d4-findings-file-shape
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-ask-sender-not-loom
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d2-key-flip-on-restart
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-post-record-order
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d3-d4-parent-re-resolved
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d6-bound-overstates-guard
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d4-no-parent-refusal-criterion
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-timeout-line-criterion
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: d5-rejected-alternative-in-record
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first review, so there is no earlier ledger and nothing recurs.
The BLOCKING finding says D2's generation rule reads a number that restarts per generation and misses a rewritten record after a goto or resume.
Five MEDIUM design findings cover the clear path reusing the ask key, the unstated deadline cap, an unreachable `no-parent`, an answer verb that names no ask, an undefined findings-file shape, and an ask sender that need not be the LOOM program.
The remaining findings are LOW or NIT: two design gaps on ordering and parent re-resolution, and local consistency and criterion fixes.
