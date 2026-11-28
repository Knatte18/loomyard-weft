---
round: 1
ledger:
  - key: merge-danger-omits-outward-records-push
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: reset-sync-failure-handover-keeps-stale-way-forward
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reset-sync-failure-outcomes-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three findings, all around the records push the reset now makes when its sync fails after the commit landed.
The change description's merge-danger section does not name that push or its effect on the run lifecycle (F1).
A hand-over on that path leaves the sync's way forward in the refusal (F2).
The branch's not-committed and hand-over outcomes have no test row (F3).
None is gating.
