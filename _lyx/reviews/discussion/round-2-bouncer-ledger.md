---
round: 2
ledger:
  - key: d5-freshness-rule-rejects-genuine-records
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-trust-claim-wrong-guard
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
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
    status: open
    class: design
    severity: MEDIUM
  - key: d6-exhausted-retry-warning-becomes-error
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
  - key: scope-overstates-d6-push-coverage
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised eight findings; round 2 raised five.
Six round 1 keys did not return and are resolved: the D6 rebase-trigger ordering and the cannot-lock-ref classification, the bounded D5 retry, the D4 failure episode, the D1 dirty-prime wait, and the goal wake list.
Two round 1 keys came back in a new form.
D5's freshness window now fails on the unstated precision of `written_at` against shed's whole-second history `At`.
D2's trust anchor now fails for a prime branch outside `require_pr_to_base`.
Three keys are new: D5's goto leaving a paused child whose notice reaches the orch, D6's exhausted-retry error turning a coalesced-push warning into a failure, and Scope overstating D6's coverage.
