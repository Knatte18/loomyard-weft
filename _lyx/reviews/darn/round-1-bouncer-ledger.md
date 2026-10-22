---
round: 1
ledger:
  - key: git-tag-format-value-skip-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: git-tag-list-option-before-double-dash-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: git-tag-writing-options-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised three LOW test-coverage findings against `TestCommandRunsMutatingGit` for the git tag read-only guard.
Each asks for table rows only, with the expected result stated in the finding.
None is gating.
