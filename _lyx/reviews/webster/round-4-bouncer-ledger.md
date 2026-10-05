---
round: 4
ledger:
  - key: escalation-record-unreadable-refusal-unspecified
    rounds: [1]
    status: resolved
  - key: loom-review-key-refusal-no-way-forward
    rounds: [2]
    status: resolved
  - key: decision-add-commit-failure-no-spec-row
    rounds: [3]
    status: resolved
  - key: decision-add-no-decisions-heading-no-way-forward
    rounds: [3]
    status: resolved
  - key: bouncer-circling-decision-refusals-no-way-forward
    rounds: [3]
    status: resolved
  - key: circling-verb-malformed-escalation-no-way-forward
    rounds: [4]
    status: open
  - key: decision-add-status-read-and-append-no-retry-clause
    rounds: [4]
    status: open
  - key: escalation-unreadable-reason-lacks-way-forward-label
    rounds: [4]
    status: open
  - key: config-review-key-spec-cell-disagrees-with-message
    rounds: [4]
    status: open
---

Every round has found a further site of `PATTERN-refusal-way-forward`: the unreadable escalation record (round 1), the `loom.yaml` review keys (round 2), three decision and circling sites (round 3), and four more in round 4.
A fresh round-4 review judged the round-3 sites compliant, so those keys are resolved.
The round-4 findings are open until a fresh review sees their fixes.
The pattern is an enumeration gap: sites reached through a shared refusal wrapper are found one per round.
