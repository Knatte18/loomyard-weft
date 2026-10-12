---
round: 1
ledger:
  - key: shortcut-group-skip-unbounded
    rounds: [1]
    status: open
    class: scope
    severity: MEDIUM
  - key: free-standing-comment-outside-member-span
    rounds: [1]
    status: open
    class: design
    severity: LOW
  - key: root-long-hand-lists-shortcuts
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
  - key: shortcutproblems-root-branch-unreached
    rounds: [1]
    status: open
    class: scope
    severity: LOW
---

Round 1 raised four findings against cards 5 and 6, none gating.
F1 (shortcut-group guard skips lack a stated bound) is the only MEDIUM, scope-class.
F2 to F4 are LOW: comment edits outside member spans, a hand-kept shortcut list in the root Long line, and an unreached resolves-to-root branch.
