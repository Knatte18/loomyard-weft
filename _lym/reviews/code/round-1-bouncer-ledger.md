---
round: 1
ledger:
  - key: heddle-set-refusal-names-wrong-flag
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: decode-way-forward-wrong-for-replay-blob
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: overview-stale-boarddir-literal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings: F1 (refusals name `--target` instead of `--targets`), F2 (`Decode` way forward wrong for a replayed blob) and F3 (`docs/overview.md` still names `BoardDir` and `_board`).
Each fix is spelled out in the finding text.
