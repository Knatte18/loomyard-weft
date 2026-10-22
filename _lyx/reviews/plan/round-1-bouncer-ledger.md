---
round: 1
ledger:
  - key: lensesfixture-leaks-new-config-fields
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
    fix: mechanical
  - key: strip-moved-units-judgment-unpinned
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unknown-selector-error-names-wrong-file
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: refusal-bound-omits-base-writes
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: emptied-fan-way-forward-wrong-first
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: golden-file-duplicates-go-layer-prose
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: testedit-wantsawscaffold-circular
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: local-file-guard-row-repeats-coverage
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: emphasis-exemption-lets-numerics-escape
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: nonscalar-selector-refusal-no-way-forward
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised ten findings, one BLOCKING and one gating design MEDIUM.
The BLOCKING one (F1) is a test-fixture leak that card 3's own gate would fail.
The MEDIUM (F2) is a weak assertion in card 5's move row.
The rest are LOW or NIT: wrong-file or missing way-forward wording in refusals, redundant tests (golden file, circular scaffold row, local-file row), and an emphasis-line exemption that skips `numerics`.
No ledger key recurs, since this is the first round.
