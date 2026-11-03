---
round: 2
ledger:
  - key: gated-expiry-stale-waiting-list-mid-turn
    rounds: [1]
    status: resolved
  - key: nudge-turn-start-guard-untested
    rounds: [1]
    status: resolved
  - key: nudge-pause-drop-and-failed-send-untested
    rounds: [1]
    status: resolved
  - key: nudge-marks-shells-named-before-send
    rounds: [1]
    status: resolved
  - key: shellwaitactive-doc-stale
    rounds: [1]
    status: resolved
  - key: awaitedshellprefixes-doc-stale
    rounds: [1]
    status: resolved
  - key: masterawaitedshellprefixes-doc-stale
    rounds: [1]
    status: resolved
  - key: nudge-queued-before-expired-turn-end
    rounds: [2]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: websterdoc-pins-output-file-count
    rounds: [2]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, three MEDIUM design and four LOW.
Round 2's review shows all seven fixed: the turn start clears the gated shell wait, the nudge marks shells named only after a successful send, the nudge paths have tests, and the three doc comments match the code.
Round 2 raised two new findings: F1, a LOW design finding that the shell nudge is queued before `expiredTurnEnd` decides, and F2, a NIT that a doc paragraph pins the count of Master's output files.
