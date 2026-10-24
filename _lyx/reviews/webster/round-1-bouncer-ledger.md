---
round: 1
ledger:
  - key: fixer-run-hung-turn-read-as-fixed
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: loomcli-doc-retired-surface
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: rework-help-text-retired-gate-names
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: envkit-retired-segment-and-gate-list
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings and no earlier ledger exists.
F1 (BLOCKING, design) is the fixer run's end derived from the closure having run instead of the gate having passed.
F2 to F4 are stale text and fixture entries naming retired rows (PR-Gate, Merge-Bouncer, PR-Rework, Merge-Review, publish gate list).
