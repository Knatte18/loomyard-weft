# batten run on a slug whose earlier run is done reports ok and does nothing

`batten-reused-slug` · Note · bug

Imported from [issue #447](https://github.com/Knatte18/loomyard/issues/447).

Seen: `lyx batten run webster-recovery --window` returned `{"existing":false,"ok":true,...}` and opened a window, but no pair was created and no loom run started.
`_lyx/shed/webster-recovery/status.json` held a batten run from 2026-10-02 for an earlier task of the same slug, in state `done` at Worktree-Teardown, so the new run read that status as finished.
The operator noticed only because no VS Code window opened.

Expected: `batten run` for a board task whose slug has a terminal status from an earlier run refuses with a clear message (or archives the old status and starts fresh), never reports ok while doing nothing.
Board slugs are reused once a task is pruned, so this will recur.

Workaround: the task was renamed to `webster-dead-ends`.
