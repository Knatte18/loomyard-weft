---
round: 1
ledger:
  - key: orientation-licenses-reading-all-state-files
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: doc-sentence-detached-from-opening
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised two consistency findings and no gating one.
F1 maps to `orientation-licenses-reading-all-state-files`, a local wording fix in the Master template and its pin.
F2 maps to `doc-sentence-detached-from-opening`, a one-sentence move in a package doc comment.
