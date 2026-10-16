---
round: 2
ledger:
  - key: d5-state-bound-not-enforced
    rounds: [1]
    status: resolved
  - key: d5-revive-pause-race
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: design-wrong
  - key: d5-enumeration-prime-orch
    rounds: [1]
    status: resolved
  - key: d6-outer-budget-no-mechanism
    rounds: [1]
    status: resolved
  - key: d6-cancellation-during-hold
    rounds: [1]
    status: resolved
  - key: d8-reexec-ignores-hub-pause-state
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: d7-matching-rule-two-readings
    rounds: [1]
    status: resolved
  - key: d3-status-read-scope-unbounded
    rounds: [1]
    status: resolved
  - key: d3-behaviors-lack-criteria
    rounds: [1]
    status: resolved
  - key: restart-verb-out-bullet-deliberation
    rounds: [1]
    status: resolved
  - key: open-risk-not-open
    rounds: [1]
    status: resolved
  - key: d5-notice-dedup-not-atomic
    rounds: [1]
    status: resolved
  - key: d5-conditioned-send-not-in-scope
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d6-cancellation-after-deadline
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised twelve findings; round 2 found none regressed and reopened two of their keys.
`d5-revive-pause-race` is reopened as BLOCKING: the round-1 bound (the pause's grace-then-interrupt) fails when the covered sessions are idle-stalled, so a revive line can run under a live pause unrecorded.
`d8-reexec-ignores-hub-pause-state` is reopened as a LOW wording fix about an entry whose grace ended in the re-exec gap.
Two keys are new: D5 presents a conditioned shuttle send as existing behavior without putting the shuttleengine change in Scope, and D6's cancellation claim is unobservable once a held call's deadline has passed.
The other round-1 keys are resolved.
