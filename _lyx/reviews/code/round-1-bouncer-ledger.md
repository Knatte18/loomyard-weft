---
round: 1
ledger:
  - key: backtick-span-swallows-prose-in-citation
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised one NIT (F1): the rewritten citation in `cmd/lyx/gitrepoboundary_test.go` backticks the ID together with a prose clause.
The fix text names the exact replacement, so it is mechanical.
There was no prior ledger to carry forward.
