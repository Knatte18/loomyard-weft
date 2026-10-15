---
round: 1
ledger:
  - key: unitpath-reader-count-false-premise
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: refusal-way-forward-wrong-binding
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: language-none-skip-unbounded
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: d6-overstates-d2-replacement
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: external-test-handle-risk-answered
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: scope-items-lack-criterion
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: testdata-exemption-narrower-than-go-ignore-set
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, one gating: the false premise that `Glyph.UnitPath` has two readers, which D4 and Scope build a spec edit on.
The rest are citation, bound, overstatement, criterion-coverage and exemption-width corrections.
