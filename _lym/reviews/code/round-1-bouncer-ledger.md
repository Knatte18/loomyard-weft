---
round: 1
ledger:
  - key: fabricengine-doc-prime-only-settings
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: beam-color-replay-normalize-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unreadable-beam-aborts-workspace-regen
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: beam-color-upsert-key-undocumented
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: lookfor-degradation-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: beam-create-colour-picker-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: beamcli-doc-omits-colour-step
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, with no prior ledger.
One gating finding is open: the missing replay and `normalizeTask` coverage for `beam_color` (F2).
The other six are doc gaps, one failure-path robustness gap and test gaps, all with a local fix their text states.
