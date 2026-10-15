---
round: 1
ledger:
  - key: bindings-header-understates-repin-trigger
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised one NIT (F1): the `bindings.go` header names only the newer-build case, while the re-pin also fires on an absent or unparseable stamp.
The review found the rest of the diff sound.
