# batten run on a slug whose earlier run is done reports ok and does nothing

`batten-reused-slug` · Note · bug, batten

Imported from [issue #447](https://github.com/Knatte18/loomyard/issues/447).

Seen: `lyx batten run webster-recovery --window` returned `{"existing":false,"ok":true,...}` and opened a window, but no pair was created and no loom run started.
`_lyx/shed/webster-recovery/status.json` held a batten run from 2026-10-02 for an earlier task of the same slug, in state `done` at Worktree-Teardown, so the new run read that status as finished.
The operator noticed only because no VS Code window opened.

Expected (operator, 2026-10-08): a slug whose task is done and pruned from the board is free, and a new task with the same slug runs as a fresh run.
A finished run's status must not stand in the way: it is archived or dropped once its task is done, so `batten run` for the new task starts at Worktree-Create.

Workaround: the task was renamed to `webster-dead-ends`.
