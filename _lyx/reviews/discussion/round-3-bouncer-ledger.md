---
round: 3
ledger:
  - key: d5-freshness-rule-rejects-genuine-records
    rounds: [1, 2]
    status: resolved
  - key: d2-trust-claim-wrong-guard
    rounds: [1, 2]
    status: resolved
  - key: d6-ignores-rebase-retry-triggers
    rounds: [1]
    status: resolved
  - key: d5-lock-refused-retry-unbounded
    rounds: [1]
    status: resolved
  - key: d4-land-failure-episode-undefined
    rounds: [1]
    status: resolved
  - key: goal-wake-list-disagrees-with-scope
    rounds: [1]
    status: resolved
  - key: d6-cannot-lock-ref-not-always-a-race
    rounds: [1]
    status: resolved
  - key: d1-dirty-prime-wait-when-skipped
    rounds: [1]
    status: resolved
  - key: d5-goto-paused-notice-reaches-orch
    rounds: [2]
    status: resolved
  - key: d6-exhausted-retry-warning-becomes-error
    rounds: [2]
    status: resolved
  - key: scope-overstates-d6-push-coverage
    rounds: [2, 3]
    status: open
    class: consistency
    severity: LOW
  - key: d5-capped-retry-no-recorded-end
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-bound-names-unchecked-property
    rounds: [3]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised eight findings, round 2 five, round 3 three.
Every gating key from rounds 1 and 2 is resolved: the freshness window and the D2 outside-list case are settled, and the paused-arm notice and exhausted-retry warning did not return.
`scope-overstates-d6-push-coverage` persists as a LOW consistency finding: Scope still omits D6's widening of the bare "(failed)" retry.
Round 3 adds two keys.
`d5-capped-retry-no-recorded-end` is a MEDIUM design finding: the retry marker has no state for a capped retry, so a later halted-arm entry can restart it.
`d2-bound-names-unchecked-property` is a LOW decision finding: D2's bound claims a PR-Gate-reviewed tip, while the Land row only reads `require_pr_to_base`.
