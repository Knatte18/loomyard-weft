---
round: 1
ledger:
  - key: d3-resumed-session-paths-undefined
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d6-dispute-evidence-misfits-discussion-write
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: q4-stale-against-d3
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all open.
D3 does not define the outcome of a master resumed in the same conversation while fork0 was in flight, and does not replace the directive's two relaunch paragraphs.
D6's dispute evidence kinds are code-level and do not fit Discussion-Write.
The support log's Q4 pick contradicts D3's per-session bound.
