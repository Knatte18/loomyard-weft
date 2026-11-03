---
round: 1
ledger:
  - key: rename-drops-result-on-record-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: help-lost-operator-contract-under-long-cap
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: record-file-name-literal-in-cli-message
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: orphan-removal-runs-for-unflagged-entry
    rounds: [1]
    status: open
    class: scope
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
F1 maps to `rename-drops-result-on-record-error`: it adds a new error type and changes the `Board.Rename` contract, so it is `design-wrong`.
F2 maps to `help-lost-operator-contract-under-long-cap`, F3 to `record-file-name-literal-in-cli-message` and F4 to `orphan-removal-runs-for-unflagged-entry`; each is a local correction the finding text already fixes.
No finding is gating at MEDIUM or worse.
