---
round: 1
ledger:
  - key: d3-pr-gate-darn-bound-false
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d10-claim-clear-lets-through-landed-or-live-run
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: d9-d11-fabric-side-names-outside-owner-set
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
  - key: d7-empty-verify-guarded-only-at-start
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: d4-rejection-record-clear-unowned
    rounds: [1]
    status: open
    class: decision
    severity: MEDIUM
  - key: d4-d8-verify-budget-and-log-retention
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: acceptance-criteria-miss-d12-and-tmux-confirmation
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
---

Round 1 raised seven findings, all open.
The gating ones are the false rejection bound on the PR-Gate to Darn edge (F1), the unbounded claim clearing in D10 (F2), fabric side names in `landingshed` and `loomcli` text (F3), and the start-only empty-verify refusal (F4).
