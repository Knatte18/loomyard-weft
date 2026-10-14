---
round: 2
ledger:
  - key: d3-resumed-session-paths-undefined
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-dispute-evidence-misfits-discussion-write
    rounds: [1]
    status: resolved
  - key: q4-stale-against-d3
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings.
Round 2 shows the D6 finding resolved: the review found the dispute evidence kinds meetable under the writer's Step 2 bound.
The D3 finding narrowed from BLOCKING to MEDIUM: the in-flight-fork0 and at-or-after-lens-spawn paths are settled, but a resume before the fork0 call and one between fork0's return and the lens spawn call have no stated outcome.
The Q4 finding persists: the support log still describes relaunch as a fresh spawn only.
