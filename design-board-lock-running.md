# board: refuse scope edits to a running task

`board-lock-running` · Note · bug, board

A task whose run has started can still be edited on the board.
On 2026-10-06 the orch folded two new issues into `bugfix-sessions` with `lyx board intake import --into` while that run sat planned and parked before Webster, so the board entry no longer matched the plan its run was executing.

`lyx board list --text` already marks a task with a run in flight (`[running · <producer>]`), but nothing refuses a write to it.

Wanted (operator, 2026-10-06): a write that changes a running task's scope (body, brief, issues, labels, a merge or an intake `into`) is refused with the reason, naming the run.
New findings go to a note of their own instead.
