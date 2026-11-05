---
round: 1
ledger:
  - key: corrupt-template-base-wedges-reconcile
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: template-base-commit-paths-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: reconcile-envelope-keys-unasserted
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orch-mail-lines-unpinned
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: followtemplate-sequence-rows-missing
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: base-read-failure-and-restore-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: reencode-folds-long-flow-mappings
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: test-split-from-doc-comment
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: docs-still-say-never-fills
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
---

Round 1 maps F1 to corrupt-template-base-wedges-reconcile, F2 to template-base-commit-paths-untested, F3 to reconcile-envelope-keys-unasserted, F4 to orch-mail-lines-unpinned, F5 to followtemplate-sequence-rows-missing, F6 to base-read-failure-and-restore-untested, F7 to reencode-folds-long-flow-mappings, F8 to test-split-from-doc-comment and F9 to docs-still-say-never-fills.
All are open.
F1 is the only finding whose fix changes an interface, since FollowTemplate's base error must be told apart from the hub file's.
