---
round: 1
ledger:
  - key: relevantpath-guard-untested
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: loomdiscussionspec-doc-says-two-files
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 raised F1 (the `RelevantPath` guard in the discussion gate resolver has no test reaching it) and F2 (the `LoomDiscussionSpec` doc comment still says two files).
Both fixes are local and spelled out in the finding text.
Neither is gating.
