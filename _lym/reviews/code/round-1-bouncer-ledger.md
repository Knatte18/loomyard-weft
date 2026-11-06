---
round: 1
ledger:
  - key: drop-notice-slot-spent-before-post
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised F1: `noticeDropLocked` in `internal/creelengine/receive.go` spends a notice slot and clears the undisclosed count before the notice is posted.
The fix is a local restore on a post error plus one test row, so it is mechanical.
No other findings were raised, and there was no earlier ledger.
