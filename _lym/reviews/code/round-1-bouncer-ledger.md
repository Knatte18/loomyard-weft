---
round: 1
ledger:
  - key: import-copies-no-named-evidence
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: design-wrong
  - key: attach-breaks-provenance-last
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: fold-gap-handling-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: path-detector-misses-forms
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: invalid-utf8-attachment-altered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: skill-help-say-evidence-absent
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: body-limit-boundary-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: close-failure-envelope-keys-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: empty-body-with-attach-unpinned
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: backtick-name-breaks-heading
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: title-rendered-with-go-escaping
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised eleven findings, F1 to F11, mapped in order to the keys above.
F1 is the only BLOCKING one: the import lists named `.lym` paths but copies no evidence.
F3 is the only gating-class MEDIUM.
The rest are local test gaps, small correctness fixes and wording fixes.
