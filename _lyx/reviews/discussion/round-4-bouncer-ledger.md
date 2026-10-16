---
round: 4
ledger:
  - key: d5-state-bound-not-enforced
    rounds: [1]
    status: resolved
  - key: d5-revive-pause-race
    rounds: [1, 2, 3, 4]
    status: open
    class: design
    severity: MEDIUM
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
    status: resolved
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
    status: resolved
  - key: d5-conditioned-send-lock-hold-unbounded
    rounds: [3, 4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-revive-sends-keys
    rounds: [4]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: d5-revive-lock-without-write-undecided
    rounds: [4]
    status: open
    class: decision
    severity: LOW
    fix: design-wrong
---

Round 1 raised twelve findings, most resolved by round 2.
`d5-revive-pause-race` is open a fourth round: a send that lands with no turn start seen lets the revived turn run under a live pause entry, and the Acceptance criterion contradicts that case.
`d5-conditioned-send-lock-hold-unbounded` returns: the two-window bound omits the ready reading before typing, the unlanded-text clear, and the uncut reed and tmux calls.
`d8-reexec-ignores-hub-pause-state` and `d6-cancellation-after-deadline` are resolved; the round 4 review examined both and rejected them as defects.
New this round: revive's "sends no key" bound is false for the guarded send it extends, and the mechanism for holding the record lock without a write is undecided.
