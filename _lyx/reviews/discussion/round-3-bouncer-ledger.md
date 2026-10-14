---
round: 3
ledger:
  - key: d3-resumed-session-paths-undefined
    rounds: [1, 2]
    status: resolved
  - key: d6-dispute-evidence-misfits-discussion-write
    rounds: [1]
    status: resolved
  - key: q4-stale-against-d3
    rounds: [1, 2]
    status: resolved
  - key: d3-reprompt-contradicts-directive-sentence
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d3-acceptance-narrows-reprompt-path
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings; round 2 resolved the D6 finding and narrowed the D3 finding to MEDIUM.
Round 3 reviewed the resume-point coverage and found it complete, and found the support log's Q4 and Turn 3 consistent with D3, so the D3 resumed-paths and Q4 keys are resolved.
Round 3 raised two new keys.
The MEDIUM design key is a directive sentence that D3's re-prompt rule contradicts; it is first raised in this round, so it is carried into the decision record's Open risks.
The LOW key is a narrow acceptance sub-item for the re-prompt path.
