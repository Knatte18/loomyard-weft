---
round: 1
ledger:
  - key: refusal-spec-rows-missing
    rounds: [1]
    status: open
    class: scope
    severity: BLOCKING
  - key: docs-constraint-package-docs
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: byte-identical-prompt-unpinned
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
  - key: await-cap-spelled-twice
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: fork-edge-bound-incomplete
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: twill-spec-test-duplicates
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: await-verb-noarg-row-duplicate
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: remove-ready-marker-test-duplicate
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: twilldirective-empty-field-refusals
    rounds: [1]
    status: open
    class: scope
    severity: LOW
  - key: ready-marker-accessor-card-cut
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: file-header-edits-outside-targets
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
---

Round 1 raised eleven findings, one BLOCKING, two MEDIUM, seven LOW and one NIT.
The BLOCKING one is the missing `contracts/specs/refusal-spec.md` rows for the reworded `discussion_producer` refusal and the new twill fan-resolution refusal.
The MEDIUM ones are the unpinned byte-identical claim for `single` and chair prompts and the package docs the docs constraint names.
The rest are test-economy and card-cut points, a duplicated cap constant, an incomplete attack-surface bound and an unlicensed guard in the directive package.
