---
round: 1
ledger:
  - key: card8-noticelog-now-missing-from-envkit-nillegal
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: card1-layout-go-untouched-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: minute-duration-format-planned-twice
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: card9-testfitoverview-per-symbol-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings against the plan.
The blocker is card 8's new nil-defaulting `NoticeLog.Now` seam, which `envkit.NilSeams` reports because `nilLegal` lacks the three nested paths.
The other three are LOW: card 1's contradictory claim about `layout.go`, the minute-duration formatter planned in both card 7 and card 9 instead of once in `statuspane`, and card 9's per-symbol `TestFitOverview`.
