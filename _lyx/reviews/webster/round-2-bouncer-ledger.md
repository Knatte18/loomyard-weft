---
round: 2
ledger:
  - key: escalation-record-unreadable-refusal-unspecified
    rounds: [1]
    status: resolved
  - key: loom-review-key-refusal-no-way-forward
    rounds: [2]
    status: open
---

Round 1 F1 (unreadable escalation record) was fixed; round 2's review found its message, spec row and reaching test in place.
Round 2 F1 is a new site of the same pattern: the `LoadConfig` refusal for the two new loom.yaml review keys lacks a way forward, a refusal-spec row and a test asserting the action.
It stays open until a fresh review sees the fix.
