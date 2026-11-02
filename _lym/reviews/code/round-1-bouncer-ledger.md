---
round: 1
ledger:
  - key: card-gate-key-bare-literal
    rounds: [1]
    status: open
    class: consistency
    severity: BLOCKING
    fix: mechanical
  - key: waitonly-marks-terminal-on-pass
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: waitthenrun-marks-terminal-on-error
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: runwithfixer-hands-refusal-to-fixer
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: card-runner-marks-terminal-before-rerun
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: writenewrun-role-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: goto-site-guard-wiring-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: unreachable-nonpositive-fixer-timeout
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: bare-verify-entry-literal
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

First round, no earlier ledger.
Round 1 raised nine findings: one BLOCKING (a third bare `"card"` gate-key literal in `decodeGates`), one MEDIUM (`waitOnly` marking the fixer record terminal after a passing earlier entry), and seven LOW or NIT.
Only the MEDIUM finding changes behavior and contract, so only it is marked `design-wrong`; the rest are local corrections whose result the finding text fixes.
