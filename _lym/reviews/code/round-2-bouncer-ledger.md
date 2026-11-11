---
round: 2
ledger:
  - key: replace-refusals-untested
    rounds: [1, 2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: add-lock-refusal-untested
    rounds: [2]
    status: open
    class: design
    severity: BLOCKING
    fix: mechanical
  - key: beam-texts-name-add-only
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: beam-run-refresh-row-vacuous
    rounds: [1]
    status: resolved
  - key: store-help-text-stale
    rounds: [1]
    status: resolved
  - key: discussion-spec-record-file-unnamed
    rounds: [1]
    status: resolved
  - key: added-entry-subset-accepts-repeat
    rounds: [1]
    status: resolved
  - key: store-lock-unpinned
    rounds: [1]
    status: resolved
  - key: history-fallback-unreached
    rounds: [1]
    status: resolved
---

Round 1 raised eight findings; round 2 clears six of them.
`replace-refusals-untested` stays open: the failed-write and failed-restore refusals of `decisionReplaceVerb` still have no test (F2).
`beam-texts-name-add-only` stays open: two boardengine doc comments still call add the only way (F3).
Round 2 adds `add-lock-refusal-untested` (F1): add's new decision-lock refusal has no test.
Both BLOCKING findings are test gaps whose fix the review text spells out.
