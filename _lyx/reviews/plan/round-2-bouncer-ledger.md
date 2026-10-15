---
round: 2
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
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card10-instructionfiles-missing-from-envkit-nillegal
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: webstercli-rundeps-never-fills-instructionfiles
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: unwrapped-start-way-forward-misfits-orch-start
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: master-engine-row-overclaims-run-and-doubles-way-forward
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: merriam-start-base-unreadable-row-goes-stale
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card8-engineless-read-states-no-bound
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: modelkey-surface-rides-in-seat-requires-card
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1's five findings do not recur in round 2, and the round 2 review's Rejected section settles each of their subjects, so all five are resolved.
Round 2 raised eight new findings.
Two are BLOCKING compile or test breaks from missed field users: card 8's `Engine` to `Engines` rename and card 10's new `InstructionFiles` field.
One MEDIUM covers `websterCLI.runDeps` not filling the new field.
Five are LOW: the orch's unwrapped way forward, the `master engine not registered` row's trigger, the stale `Merriam start base unreadable` row, card 8's missing bound, and `ModelKey` needing its own card.
