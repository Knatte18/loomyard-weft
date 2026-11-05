---
round: 1
ledger:
  - key: stale-summary-lost-on-failed-read
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: hubstrands-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: receiverpaused-worktree-choice-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: refusal-drop-replacement-failure-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: configcli-help-rewrite-unlicensed
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so there is no earlier ledger.
F1 is the only gating finding: the skip count of stale mail lives in a local variable, so a read that fails after the skip never posts the summary line.
F2 to F4 are test gaps and F5 is an unlicensed help-text hunk in `internal/configcli`.
