---
round: 1
ledger:
  - key: notice-log-now-missing-from-envkit-nillegal
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
  - key: fit-overview-per-symbol-test
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised four findings, all first seen this round.
F1 is the blocker: card 8 adds nil-defaulting `Now` fields under `NoticeLog` without registering the three paths in envkit's `nilLegal`.
F2 is a wording conflict in card 1 about `layout.go`.
F3 asks to share one minute-duration formatter in `internal/statuspane` rather than plan it in two packages.
F4 folds `TestFitOverview` into `TestRenderOverview` rows.
