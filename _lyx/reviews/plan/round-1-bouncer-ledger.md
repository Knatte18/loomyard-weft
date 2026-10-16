---
round: 1
ledger:
  - key: twin-probe-until-done-helpers
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: no-trust-inputs-duplicates-undismissable-engine
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card3-cli-row-repeats-engine-and-cli-rows
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card6-intent-omits-what-shorter-guard-admits
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card11-gate-poll-exit-conditions-unnamed
    rounds: [1]
    status: open
    class: decision
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings, all open.
F1 (card 3, duplicated `rig.probeUntilDone` helpers versus the shared `creelfake` clock) is the only gating one.
F2 and F3 are test-economy duplications in cards 10 and 3.
F4 and F5 are Intent wording gaps in cards 6 and 11.
Every proposed fix is a local correction its finding text fixes, so all are marked `mechanical`.
