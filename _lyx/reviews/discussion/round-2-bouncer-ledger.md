---
round: 2
ledger:
  - key: sandbox-list-vs-unchanged-bytes
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-forkaudit-dependency-no-fallback
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d3-bound-omits-trace-write
    rounds: [1, 2]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: start-here-confound-in-open-risks
    rounds: [1]
    status: resolved
---

Round 2 narrowed the round-1 blocker: the shared-body sandbox list is now a placement and wording defect (F3), not a contradiction with the unchanged bytes.
The D7 finding escalated: the forkaudit tool cannot reach this run's Plan-Write, and the hold exists only in prose (F1), so the fix drops D7 from this task.
The D3 bound still misstates what `card-pack` writes (F2), a local text correction.
The start-here confound drew no finding this round.
