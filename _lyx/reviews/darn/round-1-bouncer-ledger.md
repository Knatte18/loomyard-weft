---
round: 1
ledger:
  - key: orch-board-policy-keeps-message-step
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: notice-signing-split-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: notice-no-program-fallback-outside-pane
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: decision-add-help-text-overtrimmed
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings.
The gating one is the untested signing split in `decisionNoticeClient`.
The others are a leftover message step in the orch board policy, a missing program fallback when a strand claim is refused outside its pane, and help text trimmed beyond the notice paragraph.
Each fix is a local correction the finding text already fixes.
