---
round: 1
ledger:
  - key: listworktrees-unpinned-branches
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: git-version-parse-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: stale-worktree-list-failure-comment
    rounds: [1]
    status: open
    class: consistency
    severity: NIT
    fix: mechanical
---

Round 1 is the first round, with no prior ledger.
F1 maps to `listworktrees-unpinned-branches`, F2 to `git-version-parse-untested`, F3 to `stale-worktree-list-failure-comment`.
All three are local corrections whose fix text already names the result.
