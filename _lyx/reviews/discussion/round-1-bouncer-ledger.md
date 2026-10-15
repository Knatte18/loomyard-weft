---
round: 1
ledger:
  - key: d4-phase-bullet-mid-cycle-reading-premise
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: d2-decode-failure-outcome-unstated
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d3-bound-understates-strand-replacement
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised three findings, all in `decision-record.md`, none gating.
F1 (D4 phase bullet) rests on a false premise about the watcher's token reading; F2 (D2) leaves a decode failure of a present state file unassigned; F3 (D3 Bound) under-states when the runs strand is replaced.
Each review entry gives the replacement text, so each fix is a local correction.
