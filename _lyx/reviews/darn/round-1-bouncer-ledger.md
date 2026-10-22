---
round: 1
ledger:
  - key: yamlengine-entry-key-refusals-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: configsync-former-template-values-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: batcher-profile-origins-error-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW findings, all local corrections whose text fixes the result.
F1 maps to the untested entry-key refusals in `yamlengine`, F2 to the `formerTemplateValues` comment, F3 to the untested `profileOrigins` error path.
None is gating.
