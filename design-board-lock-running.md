# board: mark running tasks and refuse scope edits to them

`board-lock-running` · Note · bug, board

A task whose run has started can still be edited on the board, and nothing tells a writer that it is running.
On 2026-10-06 the orch folded two new issues into `bugfix-sessions` with `lyx board intake import --into` while that run sat planned and parked before Webster, so the board entry no longer matched the plan its run was executing.

Wanted (operator, 2026-10-06):
- The board marks which tasks are active, from the moment their run starts until it lands or is abandoned, and `lyx board list --text` and the README show the marker.
- A write that changes an active task's scope (body, brief, issues, labels, a merge or an intake `into`) is refused with the reason, naming the run.
  Where new findings go instead: a note of their own, or the next bugfix bucket.
