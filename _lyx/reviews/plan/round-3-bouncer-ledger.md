---
round: 3
ledger:
  - key: card1-breaks-webster-recover-fixtures-until-card7
    rounds: [1]
    status: resolved
  - key: run-merriambase-refusal-triggers-unreachable-behind-entry-gate
    rounds: [1]
    status: resolved
  - key: advisor-way-forward-sets-list-key-to-scalar
    rounds: [1]
    status: resolved
  - key: dropped-generic-way-forward-strips-orch-refusal
    rounds: [1]
    status: resolved
  - key: requires-rows-not-reached-through-fake-engine-set
    rounds: [1]
    status: resolved
  - key: card8-websterentry-and-battencli-stay-on-removed-engine-field
    rounds: [2]
    status: resolved
  - key: card10-instructionfiles-missing-from-envkit-nillegal
    rounds: [2]
    status: resolved
  - key: webstercli-rundeps-never-fills-instructionfiles
    rounds: [2]
    status: resolved
  - key: unwrapped-start-way-forward-misfits-orch-start
    rounds: [2]
    status: resolved
  - key: master-engine-row-overclaims-run-and-doubles-way-forward
    rounds: [2]
    status: resolved
  - key: merriam-start-base-unreadable-row-goes-stale
    rounds: [2]
    status: resolved
  - key: card8-engineless-read-states-no-bound
    rounds: [2]
    status: resolved
  - key: modelkey-surface-rides-in-seat-requires-card
    rounds: [2]
    status: resolved
  - key: card9-missing-role-entry-refusal-claim-untrue
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1's five findings and round 2's eight did not recur in round 3, whose review traced each focus item and found them clean, so all are resolved.
Round 3 raised one LOW finding: card 9's Intent says a missing role entry keeps today's refusal, which `Run` and `recoverSpawn` do not support; the gate should skip such a role.
