---
round: 1
ledger:
  - key: samplerengine-doc-paths-only-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1: `internal/samplerengine/doc.go` line 14 still said the sampler stencil carries paths only, while the new PATTERN directive makes the prompt carry the PATTERN overview.
The fix is a one-line wording change.
No other finding was raised, and no prior ledger existed.
