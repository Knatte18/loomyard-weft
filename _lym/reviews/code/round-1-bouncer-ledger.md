---
round: 1
ledger:
  - key: scratch-record-override-merges-closed-pr
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: finalize-ready-merged-pr-lacks-rule-differs
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: parent-unmerged-upstream-branch-untested
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: mechanical
  - key: retired-move-and-landing-record-closures-untested
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: orch-role-landing-halts-stale
    rounds: [1]
    status: open
    class: consistency
    severity: LOW
    fix: mechanical
  - key: ready-refusal-names-nonexistent-demand
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
  - key: repeated-retired-move-notice-drops-pr
    rounds: [1]
    status: open
    class: design
    severity: LOW
    fix: mechanical
---

Round 1 raised seven findings, three MEDIUM and four LOW, all first seen this round.
The MEDIUM ones are F1 (scratch-record override merging closed or merged pull requests), F2 (Finalize and `lym beam ready` judging a merged pull request's missing commits by different rules) and F3 (the upstream branch of `ParentUnmerged` untested).
F2 changes where a shared rule lives, so its fix needs a fresh review; the rest are local corrections the finding text already fixes.
