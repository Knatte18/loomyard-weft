---
round: 1
ledger:
  - key: gated-expiry-stale-waiting-list-mid-turn
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: nudge-turn-start-guard-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: nudge-pause-drop-and-failed-send-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: nudge-marks-shells-named-before-send
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: shellwaitactive-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: awaitedshellprefixes-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: masterawaitedshellprefixes-doc-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings (F1 to F7), none BLOCKING.
F1 to F3 are `design` at MEDIUM and gate: a gated waiting turn end keeps its list across a turn start so the bound can count Done mid-turn, and the nudge's turn-start guard, pause drop and failed send have no tests.
F4 is a LOW `design` finding: shells are marked named before the send.
F5 to F7 are LOW `consistency` findings about doc comments the diff made stale.
