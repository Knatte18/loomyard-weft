# batten run on a slug whose earlier run is done reports ok and does nothing

`batten-reused-slug` · Note · bug, batten

Imported from [issue #447](https://github.com/Knatte18/loomyard/issues/447).

Seen: `lyx batten run webster-recovery --window` returned `{"existing":false,"ok":true,...}` and opened a window, but no pair was created and no loom run started.
`_lyx/shed/webster-recovery/status.json` held a batten run from 2026-10-02 for an earlier task of the same slug, in state `done` at Worktree-Teardown, so the new run read that status as finished.
The operator noticed only because no VS Code window opened.

Expected (operator, 2026-10-08): a slug whose task is done and pruned from the board is free, and a new task with the same slug runs as a fresh run.
A finished run's status must not stand in the way: it is archived or dropped once its task is done, so `batten run` for the new task starts at Worktree-Create.

## Proposal

- `batten run` on a slug whose status is terminal and done treats the slug as free: it archives or drops the old `_lyx/shed/<slug>/` and starts at Worktree-Create.
  A halted or failed status stays, since it is meant to be resumed.
- Removing finished runs' status directories is batten's own cleanup verb, which the repo-wide cleanup command in note `lyx-cleanup` calls.
  The board never touches shed state: `lyx board prune` stays board-only.
- A running batten run's directory is never removed, so the cleanup skips any slug whose status is not terminal.

Workaround: the task was renamed to `webster-dead-ends`.
