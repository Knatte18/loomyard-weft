---
round: 1
ledger:
  - key: trace-log-claim-false-outside-worktree
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: blast-radius-omits-rc-button
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: hold-constant-comment-names-slack-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: warn-record-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: show-toggle-set-before-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
F1 (design, MEDIUM) is the only gating one: documentation claims a trace-log entry the sink cannot write outside a lym worktree.
F2 and F3 concern the change description and a comment naming less than the diff changes.
F4 and F5 are missing tests for the Warn record and for the `showToggle` seam set before wiring.
