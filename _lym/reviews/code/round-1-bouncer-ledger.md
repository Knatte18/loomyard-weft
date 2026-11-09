---
round: 1
ledger:
  - key: dotted-profile-set-no-cli-test
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: setentries-dotted-branches-unpinned
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
  - key: jsonentries-comment-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: dotted-top-level-name-undocumented
    rounds: [1]
    status: open
    class: scope
    severity: LOW
    fix: mechanical
---

Round 1 raised four LOW findings and no gating one.
F1 asks for CLI-level rows for a dotted profile set, F2 for the unpinned dotted-path branches in `TestSetEntries`,
F3 for the stale `Spec.JSONEntries` field comment, and F4 for a sentence saying a dot in a key always separates levels.
Each fix is a local correction whose result the finding text already fixes.
