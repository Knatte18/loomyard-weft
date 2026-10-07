---
round: 2
ledger:
  - key: sibling-card-cross-references-wrong
    rounds: [1]
    status: resolved
  - key: driver-silence-hidden-effect-unstated
    rounds: [1]
    status: resolved
  - key: notify-rule-escapes-recipe-blindness-scan
    rounds: [1]
    status: resolved
  - key: card1-replans-existing-compact-tests
    rounds: [1]
    status: resolved
  - key: card5-arm-tests-not-separated
    rounds: [1]
    status: resolved
  - key: cards7-8-tests-beside-existing
    rounds: [1]
    status: resolved
  - key: unused-uses-entries
    rounds: [1]
    status: resolved
  - key: runstate-pid-no-covering-test
    rounds: [2]
    status: open
    class: scope
    severity: MEDIUM
  - key: shed-waits-existing-tests-understated
    rounds: [2]
    status: open
    class: consistency
    severity: MEDIUM
  - key: notice-tests-miss-render-table
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: compact-boundary-duplicate-row
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: driver-notify-wrong-sibling-for-stop-notices
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: arm-go-stale-per-poll-comment
    rounds: [2]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised seven findings, one gating (the unbounded driver silence in cards 3 and 6).
The fresh round-2 review raised six different findings, none gating and none recurring, so every round-1 key is marked resolved.
Round 2 is two scope findings (the unpinned `RunState.PID` write, the stale `arm.go` comment) and four consistency findings (existing tests understated or duplicated in cards 1, 5 and 7, and a wrong sibling citation in card 3).
