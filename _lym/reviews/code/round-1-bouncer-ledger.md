---
round: 1
ledger:
  - key: gate-redundant-refusals-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: gate-redundant-map-build-failure-unmarked
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: research-index-omits-pilot-report
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: judgemap-doc-references-judge
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: gate-redundant-candidate-tier-zero
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: gate-redundant-module-way-forward-wording
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: pilot-report-test-count-gap
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised seven findings, one MEDIUM and the rest LOW or NIT.
The MEDIUM, `gate-redundant-refusals-untested`, is the only gating one: two refusal branches of `lym gate redundant` have no test.
Every fix is a local correction whose result the finding text already fixes, so each is mechanical.
