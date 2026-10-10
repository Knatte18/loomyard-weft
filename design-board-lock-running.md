# board: refuse scope edits to a running task

`board-lock-running` · Task · bug, board · running · Publish

A task whose run has started can still be edited on the board.
On 2026-10-06 the orch folded two new issues into `bugfix-sessions` with `lyx board intake import --into` while that run sat planned and parked before Webster, so the board entry no longer matched the plan its run was executing.

The status already exists and is stored on the entry: the loom run writes `status` on every transition (`SetBoardStatus` in `internal/loomcli/wiring.go`, through `boardStatus`), such as `running · Webster`, and landing sets `done`; `lyx board list --text` shows it.
Nothing refuses a write to an entry whose status is set.
The operator wants the lock to start when a task is claimed (operator, 2026-10-06): the status is set at once when its run starts, before Discussion, so no edit slips in between the start and the first transition.

Wanted (operator, 2026-10-06): a write that changes a running task's scope (body, brief, issues, labels, a merge or an intake `into`) is refused with the reason, naming the run.
New findings go to a note of their own instead.
