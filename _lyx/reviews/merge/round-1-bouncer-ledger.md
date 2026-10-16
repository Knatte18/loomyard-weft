---
round: 1
ledger:
  - key: addstrand-stale-anchor-workaround
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding (F1): `addStrand` in `internal/creelcli/smoke_helpers_test.go` keeps a `--anchor own-window` workaround and a comment that the parent's merged anchor-default change made stale.
The fix is local and fully specified by the finding text.
