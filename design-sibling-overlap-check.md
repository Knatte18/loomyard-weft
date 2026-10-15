# Check overlap between tasks that wait on the same entry

`sibling-overlap-check` · Note · enhancement, board, batten, orch

## Seen

On 2026-10-15, four tasks had `depends_on` `kickstart-per-card`: `webster-master-no-code`, `plan-test-file-path`, `lighter-test-rules` and `engine-registry`.
The newly-ready auto-start would have started all four together, though the first three edit the same Webster, plan and stencil files and their tests.
The operator noticed just before the landing; the orch then chained the three.
The orch checks a new task against the running ones only, never against other tasks waiting on the same entry, and each task got its `depends_on` at a different time.

## Idea

Catch it when it is cheap, in two places:
- `lyx board set-deps` and `upsert` warn when the new `depends_on` makes a task a sibling of another task with a shared module label (and later the paths of note `labels-from-diff`).
- Batten newly-ready start: when a landing makes several tasks ready at once, start one per overlapping group and leave the rest for the orch, naming the overlap in the notice.
The orch role also gains the rule to check siblings whenever it sets `depends_on`.