---
round: 1
ledger:
  - key: doc-go-nongo-file-targets
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: test-tier-unstated-way-forward
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: cards-8-9-redundant-rows
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: mechanical
  - key: card4-new-test-file-via-handles
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card8-test-pseudo-unit-row
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card8-seam-reader-halt-no-actor
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card6-external-struct-reflection
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: design-wrong
  - key: card4-tracked-root-head-narrowing
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: card4-trailing-paren-args
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: resolvetargets-doc-consumer-list
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: wirelightweight-doc-planindex
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
  - key: resolveglyphs-empty-list-unreachable
    rounds: [1]
    status: open
    class: design
    severity: NIT
    fix: mechanical
---

Round 1 raised 12 findings, none BLOCKING.
The only gating one is `test-tier-unstated-way-forward` (F2), whose fix extends a refusal's way forward and so needs a fresh review.
`card6-external-struct-reflection` (F7) adds a new bound to the scan and is marked `design-wrong` for the same reason.
The rest are local corrections the finding text already fixes.
