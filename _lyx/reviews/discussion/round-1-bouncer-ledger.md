---
round: 1
ledger:
  - key: fan-setting-null-recording-undefined
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: darn-review-entry-model-sources
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: status-and-build-budget-shared-source
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: null-fan-contradicts-shape-refusal
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: envkit-setting-fake-left-open
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: pattern-resolved-value-home-outside-review
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: snapshot-rationale-parity-claim
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
  - key: negative-budget-refusal-key-name
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: constraints-bullet-restates-criteria
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 raised nine findings: one MEDIUM design finding (a null or empty fan setting leaves fan recording undefined while the fix row requires a recorded round), five LOW and three NIT.
The MEDIUM finding gates and adds a rule to a decision, so it needs fresh review.
The shared-budget-source finding also adds requirements to D8 and is treated as design-wrong.
The rest are local text corrections whose result the finding already fixes.
