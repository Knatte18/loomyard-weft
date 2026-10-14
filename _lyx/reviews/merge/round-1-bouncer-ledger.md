---
round: 1
ledger:
  - key: null-tag-vs-scalar-or-list-doc
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised F1, mapped to `null-tag-vs-scalar-or-list-doc`: the `SetValues` and `SetSparse` doc comments state the null-tag rule for every scalar, but a key at a scalar-or-list open-map path always becomes a string.
The fix is a local rewording of one doc-comment sentence in two files.
No prior ledger existed, so nothing carries forward.
