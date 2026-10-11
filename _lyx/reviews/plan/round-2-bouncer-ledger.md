---
round: 2
ledger:
  - key: parseseed-test-economy-repeat
    rounds: [1]
    status: resolved
  - key: shed-span-timestamp-and-absent-seed-untested
    rounds: [1]
    status: resolved
  - key: all-runs-sum-and-paragraph-unpinned
    rounds: [1]
    status: resolved
  - key: shedrun-doc-claim-vs-treerunfilerels
    rounds: [1, 2]
    status: open
    class: consistency
    severity: LOW
  - key: last-at-parsed-only-on-folded-entry
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
  - key: run-id-error-unwrapped-vs-decision-9
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: resolvehub-cwd-parent-vs-worktree-parent
    rounds: [2]
    status: open
    class: decision
    severity: LOW
  - key: archivedslugs-missing-commit-skip-untested
    rounds: [2]
    status: open
    class: consistency
    severity: LOW
---

Round 1's gating test-economy finding, the missing timestamp and absent-seed test inputs, and the unpinned "All runs" sums are resolved: round 2's focus verdicts found each covered.
The shedrun doc claim persists as F1 in a narrower form: `SeedRel`, `StatusRel` and `DriveReportsRel` still contradict the claim, and the `paths.go` header omits the tree-relative kind.
Round 2 adds four LOW findings on card 3, card 4 and card 5 (F2 to F5), none gating.
