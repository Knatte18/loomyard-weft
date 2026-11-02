---
round: 1
ledger:
  - key: card9-ends-fixers-on-unmoved-run
    rounds: [1]
    status: open
    class: design
    severity: MEDIUM
    fix: design-wrong
  - key: runproc-doc-lacks-fixer-roles
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: card3-stale-headers-naming-deleted-members
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
  - key: recipe-spec-lacks-fixer-element-key
    rounds: [1]
    status: open
    class: consistency
    severity: MEDIUM
    fix: mechanical
---

Round 1 raised four MEDIUM findings.
The one gating finding is card 9 ending live fixers on a goto that is refused or a relaunch that does not move the run.
The other three are consistency gaps: runproc's doc role list, stale comments after card 3's deletions, and the recipe spec's `fixer` element key.
