---
round: 1
ledger:
  - key: card5-untagged-seam-spawns-go-list
    rounds: [1]
    status: open
    class: design
    severity: BLOCKING
  - key: card5-per-identifier-fallback-no-mechanism
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: card4-import-selector-exclusion-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
---

Round 1 raised three findings.
F1 (`card5-untagged-seam-spawns-go-list`) is BLOCKING: `defaultTypesLoader` is replaced only in `internal/planglyph`'s `TestMain`, so webstercli's untagged `TestValidateCmd_Envelopes` reaches the real `go list`.
F2 (`card5-per-identifier-fallback-no-mechanism`) is MEDIUM: the re-signed `scanReferences` takes whole files, so D2's per-identifier fallback has no stated mechanism.
F3 (`card4-import-selector-exclusion-unbounded`) is LOW: card 4's import-selector exclusion drops method matches on shadowing locals with no stated bound.
