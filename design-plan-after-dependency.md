# A module overlap lets a task run Discussion at once and wait only before Plan

`plan-after-dependency` · Note · feature, board, batten, loom

## Seen (operator, 2026-11-01)

Two tasks that share a module label are chained with `depends_on` today, so the later one waits for the earlier to land before it starts at all, Discussion included.
Finer labels or file-level overlap checks after Plan were weighed and rejected: what a task touches is unknown until it is planned, two plans made at once against moving code do not match, and pausing runs before Webster by what they planned is too complex.
Only Discussion can usefully run alongside; planning must be done against code that does not change under it.

## Idea

A second kind of dependency beside `depends_on`:
- `depends_on`, as today: the task does not start until the named entries land; for a real dependency, where Discussion needs the other's result.
- `plan_after`: the task starts and runs Discussion at once, then waits before Plan until the named tasks have landed; for tasks that only share a module.
- The overlap notice of `set-deps` and the upsert verbs proposes `plan_after` instead of `depends_on`.
- For darn, which has no Discussion or Plan of its own, `plan_after` acts as `depends_on`.

## Waits on

Note `discussed-task-split` first: the decision record on the board, so a discussed task waiting before Plan keeps its record where the board shows it (operator, 2026-11-01).
