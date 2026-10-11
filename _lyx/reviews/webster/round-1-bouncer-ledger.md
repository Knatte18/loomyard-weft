---
round: 1
ledger:
  - key: parserless-engine-slash-waits-submit-window
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
  - key: stale-session-start-line-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: pending-call-window-widening-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised three findings.
F1 (`parserless-engine-slash-waits-submit-window`): `/clear` and `/compact` on an engine without `SessionSignalParser` wait out the whole submit window, against the plan's no-capability-unchanged decision.
F2 (`stale-session-start-line-untested`): no test row seeds a matching session-start line before the pre-send offset.
F3 (`pending-call-window-widening-untested`): the backward window widening in `holdsPendingToolCall` has no test.
No earlier ledger exists.
