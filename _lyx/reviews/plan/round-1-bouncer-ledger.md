---
round: 1
ledger:
  - key: noticelog-now-seam-missing-from-envkit-nillegal
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: card1-layout-go-untouched-contradiction
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: minute-duration-format-planned-twice
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: testfitoverview-per-symbol-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 raised four findings, all open.
F1 maps to the BLOCKING key: card 8's `NoticeLog.Now` seam needs three `nilLegal` paths in envkit.
F2 maps to the card 1 `layout.go` wording conflict.
F3 maps to the duplicated minute-duration formatter across cards 7 and 9.
F4 maps to card 9's per-symbol `TestFitOverview`.
