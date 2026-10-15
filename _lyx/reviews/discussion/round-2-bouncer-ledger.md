---
round: 2
ledger:
  - key: d8-carrier-task-summary-false-premise
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pattern-read-set-three-wordings
    rounds: [1]
    status: resolved
  - key: empty-pattern-directive-criterion-unmeetable
    rounds: [1]
    status: resolved
  - key: d3-replaced-span-ambiguous
    rounds: [1]
    status: resolved
  - key: d1-read-ban-bound-unstated
    rounds: [1]
    status: resolved
  - key: d3-fragment-claims-rules-current
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: goal-pattern-read-set-wider-than-d1
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: d3-rest-unchanged-wrong-for-plain-fragment
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1's carrier finding is reopened in round 2 as F1: the board-note carrier D8 now names is never handed D8, and an agent-approved landing has no orch turn.
Its fix changes the carrier, so it needs a fresh review.
The other four round 1 findings were fixed and no longer appear; round 2's Rejected section confirms the read set, replaced spans and empty-directive criterion.
Round 2 adds three local wording findings: D3's "rules stay current" claim against the Open risks, the Goal's wider pattern read set, and D3's "rest unchanged" sentence for the plain fragment.
Each has its replacement text in the finding, so each is a mechanical fix.
