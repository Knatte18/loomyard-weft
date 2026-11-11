---
round: 1
ledger:
  - key: replayer-unversioned-sides-untested
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: upgradestore-dedup-and-keep-version-rows-missing
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised two LOW scope findings, both test-coverage gaps in `internal/heddleengine`.
F1 maps to `replayer-unversioned-sides-untested`; F2 maps to `upgradestore-dedup-and-keep-version-rows-missing`.
Neither is gating, so the loop converged.
