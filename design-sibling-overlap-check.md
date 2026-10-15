# Warn on overlap between tasks that wait on the same entry

`sibling-overlap-check` · Task · enhancement, board · running · Worktree-Create

## Seen

On 2026-10-15, four tasks had `depends_on` `kickstart-per-card`: `webster-master-no-code`, `plan-test-file-path`, `lighter-test-rules` and `engine-registry`.
The newly-ready auto-start would have started all four together, though the first three edit the same Webster, plan and stencil files and their tests.
The operator noticed just before the landing; the orch then chained the three.
The orch checks a new task against the running ones only, never against other tasks waiting on the same entry, and each task got its `depends_on` at a different time.

## Scope

Operator decision 2026-10-15: build both parts below.

- `lyx board set-deps` and `upsert` warn when the new `depends_on` makes a task a sibling of another task with a shared module label, that is another task waiting on one of the same entries.
  The warning names the sibling and the shared labels; the write still succeeds.
  Path-based overlap waits for note `labels-from-diff`.
- The rule to check siblings, not only running tasks, whenever `depends_on` is set goes into the `ly:board` skill beside its `depends_on` guidance, since every agent that writes the board loads that skill.
  The orch role needs no copy.

Out of scope: batten starting one task per overlapping group at a newly-ready start, which is note `batten-ready-overlap`.
