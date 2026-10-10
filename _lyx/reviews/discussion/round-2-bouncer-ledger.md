---
round: 2
ledger:
  - key: d3-pr-gate-darn-bound-false
    rounds: [1]
    status: resolved
  - key: d10-claim-clear-lets-through-landed-or-live-run
    rounds: [1, 2]
    status: open
    class: design
    severity: MEDIUM
  - key: d9-d11-fabric-side-names-outside-owner-set
    rounds: [1]
    status: resolved
  - key: d7-empty-verify-guarded-only-at-start
    rounds: [1]
    status: resolved
  - key: d4-rejection-record-clear-unowned
    rounds: [1]
    status: resolved
  - key: d4-d8-verify-budget-and-log-retention
    rounds: [1]
    status: resolved
  - key: acceptance-criteria-miss-d12-and-tmux-confirmation
    rounds: [1]
    status: resolved
  - key: d1-arm-builds-loom-shed-for-darn-run
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: d6-call-time-stop-no-way-forward-or-spec-row
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
  - key: d4-d7-verify-attempts-guarded-only-at-start
    rounds: [2]
    status: open
    class: design
    severity: MEDIUM
---

Round 1's findings on the PR-Gate bound, the fabric side names and the start-only empty-verify refusal are settled in round 2; the review names the first two explicitly and the D6 reader as covering the third.
Round 1's remaining LOW and decision findings were not raised again and are treated as resolved.
D10 is the one carried key: the clear now fails on the `lyx fabric remove` route for a landed run whose done mark failed (F4).
Round 2 raised three new design findings: the arm builds loom's shed for a darn seed (F1), the D6 call-time stop has no way forward or spec row (F2), and `verify_attempts` below 1 is refused only at start (F3).
