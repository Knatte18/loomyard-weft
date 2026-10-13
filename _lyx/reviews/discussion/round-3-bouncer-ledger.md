---
round: 3
ledger:
  - key: d5-state-bound-not-enforced
    rounds: [1]
    status: resolved
  - key: d5-revive-pause-race
    rounds: [1, 2, 3]
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
    rounds: [1, 2, 3]
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
    status: resolved
  - key: d6-cancellation-after-deadline
    rounds: [2, 3]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: d5-conditioned-send-lock-hold-unbounded
    rounds: [3]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
---

Round 1 raised twelve findings, most resolved by round 2.
`d5-revive-pause-race` has been open three rounds: the lock fix of round 2 closes the gap before the send but not the gap between a landing and the hook-written turn start, so an idle-stalled hub still lets a revived turn run unpaused and unrecorded.
`d8-reexec-ignores-hub-pause-state` returns a third time as a LOW wording fix about the watcher's memory-only handled set.
`d6-cancellation-after-deadline` is narrowed to a qualifier on the cancelled-during-hold sentence and criterion once the deadline has passed in the hold.
`d5-conditioned-send-not-in-scope` is resolved; the conditioned send is now named in Scope In.
New this round: the conditioned send's lock-hold duration is unbounded when it falls into the idle wait.
