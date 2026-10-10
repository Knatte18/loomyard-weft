---
round: 3
ledger:
  - key: d3-pr-gate-darn-bound-false
    rounds: [1]
    status: resolved
  - key: d10-claim-clear-lets-through-landed-or-live-run
    rounds: [1, 2]
    status: resolved
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
    status: resolved
  - key: d6-call-time-stop-no-way-forward-or-spec-row
    rounds: [2, 3]
    status: open
    class: design
    severity: BLOCKING
  - key: d4-d7-verify-attempts-guarded-only-at-start
    rounds: [2, 3]
    status: open
    class: consistency
    severity: NIT
  - key: darn-reentry-input-missing
    rounds: [3]
    status: open
    class: design
    severity: MEDIUM
  - key: reject-not-recipe-neutral-for-darn
    rounds: [3]
    status: open
    class: consistency
    severity: LOW
---

Round 3 closes the D10 claim-clear key and the arm-builder key: the review's focus verdicts call both not a defect.
The refusal gap carried from round 2 (`d6-call-time-stop-no-way-forward-or-spec-row`) is reopened at BLOCKING as F1 and widened to every refusal the design adds or rewords: the spent `Darn` verify budget, D9's commit failure, Publish's verify-failed clause and the gate-slot row.
The `verify_attempts` guard key persists only as F4, a wording inconsistency across D1, D4 and D7.
New in round 3: the `Darn` re-entry input (F2, design MEDIUM) and `reject`'s darn handling (F3, consistency LOW).
