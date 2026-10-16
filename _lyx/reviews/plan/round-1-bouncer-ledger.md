---
round: 1
ledger:
  - key: continue-bound-omits-provider-reliance
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: teststop-duplicates-continue-after-stop-row
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised two scope findings and no gating ones.
F1 asks card 3's `Bound:` sentence and card 2's `Watcher.recordSessionID` bullet to state the provider reliance D3 records.
F2 asks card 4 to drop its `TestStop` extension, which card 3's stop-then-continue row already pins.
Both fixes are fully specified in the finding text.
