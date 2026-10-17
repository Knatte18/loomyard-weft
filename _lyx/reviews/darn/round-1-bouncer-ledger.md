---
round: 1
ledger:
  - key: doc-go-retired-no-git-fork-rule
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised one finding: the older fork-discipline paragraph in `internal/burlerengine/doc.go` still says forks run no git commands, against the paragraph the diff adds.
The fix text names the exact replacement wording, so it is mechanical.
No earlier ledger exists.
