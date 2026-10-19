---
round: 1
ledger:
  - key: bar-line2-time-segment-duplicated
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
---

Round 1 raised F1: `statusFormatSessions` in `internal/reedengine/bar.go` ends with the right-aligned time segment twice, and no test counts it.
The review gives the exact fix, dropping the second segment and adding a count assertion, so it is mechanical.
The review's rejected items and focus-area results raised nothing else.
