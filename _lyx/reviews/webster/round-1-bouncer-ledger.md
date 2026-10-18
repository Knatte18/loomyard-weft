---
round: 1
ledger:
  - key: orchdirname-comment-second-use
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised F1 only: the `orchDirName` doc comment in `internal/orchcli/paths.go` omits its second use under `_lyx`.
The fix is a local comment edit whose text the finding gives.
