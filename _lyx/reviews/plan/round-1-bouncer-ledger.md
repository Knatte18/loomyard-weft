---
round: 1
ledger:
  - key: card7-dialog-notice-guard-branches-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: card8-live-scenario-command-left-open
    rounds: [1]
    status: open
    class: decision
    severity: LOW
---

Round 1 raised two findings, both non-gating.
F1: card 7's no-reader and pane-too-short guards have no `TestWait_DialogNotice` row that reaches them.
F2: card 8 calls its `rm -f ./smoke-ask-$$` command only the first candidate, leaving the live scenario's command open.
