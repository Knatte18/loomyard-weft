---
round: 1
ledger:
  - key: judgesfirst-empty-reason-on-status-stop
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: pawl-escalate-claim-unsettled
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: escalate-loop-spends-shared-arbiter-budget
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: new-prefix-must-avoid-decision-halt-prefix
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings on `decision-record.md`.
The blocking one is D4 handing `JudgesFirst` an empty reason on a stop rebuilt from a status.
The pawl escalate claim in D4 is the second gating finding.
The shared arbiter budget in D4's Bound and the decision halt prefix in D3 are LOW wording fixes.
