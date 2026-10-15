---
round: 1
ledger:
  - key: d3-multiline-clear-undecided
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-post-settle-restore-timing
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d1-bound-counts-caret-cells
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d5-start-match-by-time
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d2-progress-line-stream
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: acceptance-criteria-gaps-d2-d8
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first review of the discussion pair.
F1 and F2 are the gating findings, both about D3's interrupt clear: the clear sequence is required but undecided, and the single post-settle read rests on an unstated timing assumption.
F4 is a design finding at LOW about D5's start-header selection.
F3, F5 and F6 are local consistency corrections.
