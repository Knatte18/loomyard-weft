# Fixer report — discussion round 1

## Fixed

- **F1 (BLOCKING), index-only removal leaves records on disk.**
  `decision-record.md`: Decision 1 now makes the drop part of creating the new branch, so the parent's records never reach disk.
  Constraints gained a "No run records on disk" bullet stating the requirement and why an index-only removal after `git worktree add -b` fails it.
  The Fabric Destruction Chokepoint bullet now names the two mechanisms that satisfy it: a no-checkout fork that keeps the subtree off disk, or a working-tree delete through `destroy.go`.
  `support-log.md`: Interview turn 6 and Question ledger #6 updated to match.
  A new Rejected alternatives entry covers index-only removal after a normal checkout.
- **F2 (MEDIUM), the adopt path can adopt an unmodified fork.**
  Decision 2's rationale is narrowed to weft branches that survive `Remove` and `Add`'s rollback.
  A new Open risk names the crash window between the fork and the origin commit, the manual recovery that reaches the adopt path, and the remedy.
  A new Rejected alternatives entry records why the adopt path does not drop for such a fork.
- **F3 (LOW), no test anchor for the single-commit rule.**
  Decision 5 fixes the message to today's `fabric: record parent branch for <slug>`.
  Acceptance criterion 1 now requires the new weft branch to be exactly one commit ahead of the fork point.
  Question ledger #5 updated to match.
- **F4 (NIT), dependency-list fact in Decision 4.**
  Removed the `shedrun` import-list sentence; support-log Interview turn 5 keeps the cycle check.

`lyx loom validate-discussion` exits 0 on the edited pair.

## Deferred

None.
