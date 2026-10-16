---
round: 2
ledger:
  - key: fan-shape-error-no-way-forward-or-spec-row
    rounds: [1]
    status: resolved
  - key: bouncer-cluster-fans-ignores-recipe-fan
    rounds: [1]
    status: resolved
  - key: twill-engine-refusal-two-ways-forward
    rounds: [1]
    status: resolved
  - key: unrecorded-names-refusal-archives-before-probe
    rounds: [1]
    status: resolved
  - key: single-fixer-keeps-stale-twill-lenses-and-forks
    rounds: [1]
    status: resolved
  - key: template-self-consistency-test-pins-stale
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: new-lens-emphasis-sentence-unresolved
    rounds: [2]
    status: open
    class: decision
    severity: MEDIUM
    fix: design-wrong
  - key: configreg-local-denied-test-omitted
    rounds: [2]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: fan-shape-error-list-entry-branch-untested
    rounds: [2]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised five findings against cards 3, 5, 6, 7 and 8; round 2's review rejected every focus item covering them, so all five are resolved.
Round 2 raised four new findings.
F1 is card 1 leaving `TestConfigTemplate_DecodesAndIsSelfConsistent` unedited though it pins the template's lens and fan counts.
F2 is the emphasis sentence that test forces on lenses whose bench text ends otherwise, which card 1 leaves unruled against D4 and D5.
F3 is card 8 omitting the configreg test that pins loom's local-denied keys.
F4 is the list-entry branch of `fanShapeError` having no test input.
Both BLOCKING findings are tests that pin a count or list the cards change; the pattern suggests checking the other tests in the packages the cards touch.
