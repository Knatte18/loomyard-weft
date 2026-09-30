---
round: 1
ledger:
  - key: index-only-drop-leaves-records-on-disk
    rounds: [1]
    status: resolved
  - key: adopt-path-crash-window-unmodified-fork
    rounds: [1]
    status: resolved
  - key: single-commit-rule-no-test-anchor
    rounds: [1]
    status: resolved
  - key: decision4-import-list-fact
    rounds: [1]
    status: resolved
---

Round 1 is the first round; there is no prior ledger.

The review raised one blocking finding: the Fabric Destruction Chokepoint constraint offered an index-only removal as an escape, but `Add`'s `git worktree add -b` checks the parent's `_lyx/shed/` out first, so the records would stay on disk and the bug would persist.
The fixer added a "No run records on disk" constraint, named the two valid mechanisms (a no-checkout fork, or a delete through `destroy.go`), and left the choice to Plan.

Three smaller findings followed: Decision 2's "an existing branch is the pair's own" ignored a crash between fork and origin commit, now an Open risk with its remedy;
Decision 5 left the commit message open and criterion 1 did not pin the single commit, now both fixed;
Decision 4 carried a dependency-list fact the writer stencil forbids, now removed.

All four are resolved as of round 1.
Carried as a non-ledger observation: Decision 1's "never reach disk" wording is stronger than the Constraints bullet, which matters only if Plan picks the delete mechanism.
