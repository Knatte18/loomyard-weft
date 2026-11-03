---
round: 1
ledger:
  - key: summary-claims-no-beam-change-under-main-reads
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding, F1, mapped to `summary-claims-no-beam-change-under-main-reads`.
The summary's closing sentence of "How the two sides met" overstates: the beam extended `fabricengine.ConfigDirs` with a `Records` field that main's hibernate code reads, though harmlessly.
The fix is a local rewording of that sentence, with the replacement text given in the finding.
There was no earlier ledger, so nothing carries forward.
