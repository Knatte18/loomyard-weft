---
round: 1
ledger:
  - key: replace-refusals-untested
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: beam-run-refresh-row-vacuous
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: store-help-text-stale
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: discussion-spec-record-file-unnamed
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: added-entry-subset-accepts-repeat
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: beam-texts-name-add-only
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: store-lock-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: history-fallback-unreached
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
---

Round 1 raised eight findings, F1 to F8, mapped in order to the keys above.
F1 is the only BLOCKING one: refusals added by `decisionReplaceVerb` and `landReplace` lack tests.
F2 is the only gating design finding at MEDIUM.
F5 and F8 change an algorithm and a function boundary, so their fixes need a fresh review.
