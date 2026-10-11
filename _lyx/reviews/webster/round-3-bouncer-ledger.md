---
round: 3
ledger:
  - key: parserless-engine-slash-waits-submit-window
    rounds: [1]
    status: resolved
  - key: stale-session-start-line-untested
    rounds: [1]
    status: resolved
  - key: pending-call-window-widening-untested
    rounds: [1]
    status: resolved
  - key: integration-driver-bootstrap-stub-no-session-start
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
  - key: smoke-driver-strand-stub-no-session-start
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
  - key: smoke-start-tail-shares-driver-stub
    rounds: [3]
    status: open
    class: scope
    severity: BLOCKING
---

Rounds 1 and 2 closed the three round-1 findings.
Round 3 found that card 7's `RecordsSessionStarts` startup gating broke the stub providers in `internal/loomcli` tests, which never record a `SessionStart` line (F1, F2, F3).
F1 and F2 are the two stub helpers; F3 is covered by F2's helper.
