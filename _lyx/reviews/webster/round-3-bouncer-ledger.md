---
round: 3
ledger:
  - key: escalation-record-unreadable-refusal-unspecified
    rounds: [1]
    status: resolved
  - key: loom-review-key-refusal-no-way-forward
    rounds: [2]
    status: resolved
  - key: decision-add-commit-failure-no-spec-row
    rounds: [3]
    status: open
  - key: decision-add-no-decisions-heading-no-way-forward
    rounds: [3]
    status: open
  - key: bouncer-circling-decision-refusals-no-way-forward
    rounds: [3]
    status: open
---

Each round has found a new site of `PATTERN-refusal-way-forward`: the unreadable escalation record (round 1), the `loom.yaml` review keys (round 2), and three more in round 3.
The first two were fixed and a fresh review found them compliant.
The three round-3 findings are open until a fresh review sees their fixes.
