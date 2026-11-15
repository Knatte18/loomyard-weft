---
round: 1
ledger:
  - key: session-stop-cases-unpinned-by-wait-test
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised F1: `TestPRReview_Wait` has no row asserting the session is stopped for a recorded `ReviewEnded`, or for Continue on a clean tree at the gated head.
The finding text names the rows to extend and the assertion to use, so the fix is a local test correction.
