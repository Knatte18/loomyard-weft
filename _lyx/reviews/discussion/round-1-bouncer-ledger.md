---
round: 1
ledger:
  - key: legacy-done-fold-reverts-newer-status
    rounds: [1]
    status: resolved
  - key: tier-rule-exempt-done-dependencies
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-miss-committed-defaults
    rounds: [1]
    status: resolved
  - key: notes-aliases-behavior-change-unstated
    rounds: [1]
    status: resolved
  - key: support-log-contradicts-decision-21
    rounds: [1]
    status: resolved
  - key: bolt-naming-rule-in-goal-not-constraints
    rounds: [1]
    status: resolved
---

Round 1 was the first round, so there was no earlier ledger to carry forward.
The reviewer raised six findings, and the fixer applied all six in the same round.
The most important one was the legacy done-mark fold, which would have re-applied historical done marks forever.
It is fixed by a `legacy_done` slug record that migration seeds and `retire-legacy` drops.
The other five were smaller:
done dependencies are now exempt from the tier rule, the acceptance criteria cover the creation and `abandoned` defaults, Decision 13 states the alias behavior, the support log is reconciled with the hub's replacement of Decision 21, and the "never Bolt" naming rule moved from Goal to Constraints.
All six are resolved, and nothing is open.
