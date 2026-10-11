---
round: 1
ledger:
  - key: card2-stale-transcript-only-activity-docs
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised one finding, F1, mapped to `card2-stale-transcript-only-activity-docs`.
Card 2 rewrites some activity docs but leaves the `ActivityReader.TurnEndActivity` doc, the `Claude.TurnEndActivity` doc and the `claudeengine/activity.go` file header on the old transcript-only contract.
It is non-gating.
