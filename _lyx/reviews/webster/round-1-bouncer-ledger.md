---
round: 1
ledger:
  - key: stale-pointer-to-deleted-bare-vocabulary-predicate
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised F1, a comment in `internal/fabricengine/mergevocab_test.go` that still names the deleted `bareVocabularyToken`.
It is the only open key, and there was no earlier ledger to carry forward.
