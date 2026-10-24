# Fixer report — plan, round 2

## Changes

- **F1** (fixed as proposed):
  - Card 35's Intent now ends the pre-step-1 root-pair refusal with a way forward: a project entry through `lyx board upsert` under a slug of its own, then `lyx beam create` on it, the same route as the never-started refusal.
  - Card 8's step 1 gives the empty-`ParentBranch` refusal the same clause.
  - In both cards, the `TestPromote` and `TestResetRecordsToParent` prose now asserts that the refusal text carries both `lyx board upsert` and `lyx beam create`, not only the words root pair.
  - Only Intent and test prose changed, and no target or verify line moved.
- **F2** (fixed as proposed):
  - `TestMarkBeam` in card 2 gains a row for a pair whose records worktree the test removes. The row is refused naming `lyx fabric reconcile` with no commit made, and sits beside the deleted-record row.
  - `TestResetRecordsToParent` in card 8 gains the same row, placed beside its deleted-record row.
  - Only test prose changed.

## Self-check

- fix-consistency: no findings.
- fix-claims: no findings.
- fix-proposal: no findings.
- fix-affected-set: no findings.

## Disputed

None.

## Deferred

None.
