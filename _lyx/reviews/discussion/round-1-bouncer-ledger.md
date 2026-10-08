---
round: 1
ledger:
  - key: d1-earlier-round-source-unspecified
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-entry-identity-across-generations
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-commit-route-plan-review
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d2-write-edge-unbounded
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: goal-omits-round-floor
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: carry-over-meaning-only-in-support-log
    rounds: [1]
    status: open
    class: decision
    severity: LOW
  - key: auto-mode-assumptions-restate-decisions
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 is the first round; there is no earlier ledger.
Three `design` findings at MEDIUM gate the loop: D1's source for "open in an earlier round", D2's entry identity across generations, and the missing commit route for Plan-Review.
The other four are non-gating: a missing bound on D2, a Goal that omits the round floor, a carry-over meaning kept only in the support log, and repeated reasoning in the Auto-mode assumptions.
