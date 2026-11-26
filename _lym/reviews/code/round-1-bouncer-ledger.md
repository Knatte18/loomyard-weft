---
round: 1
ledger:
  - key: map-format-version-never-read
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: other-package-unseen-tests-no-caveat
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
    fix: design-wrong
  - key: caveat-assembly-no-text-assertion
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: unresolved-no-coverage-wrong-cause
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: untiered-and-undeclared-tier-no-caveat
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: index-version-mismatch-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: judgeruns-doc-stale-reason
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: merge-danger-omits-cross-binary-formats
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 is the first round, so every key is new and open.
The review found that the diff removes the miss the board entry reports, with gaps around it:
maps an older binary wrote are not remapped, and the reach caveats leave some unseen tests unnamed or named under the wrong cause.
The remaining keys concern missing test assertions, one doc comment and the change description's `## Merge danger` section.
