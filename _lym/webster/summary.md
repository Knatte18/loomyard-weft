# Board viewer, its server and the VS Code keys

All batches of the plan were already recorded `done` when this session started, so it began no batch and spawned no fork.
This summary rests on the progress trail and the plan's overview.
This session read no batch digest, so it has no scope guard result: the deviations and renames of each batch are not known to it.

## 01-board-view-projection

Built the board view projection `board-data.js` with entry and board revisions, the stale-revision refusal on writes,
the embedded `board.html` page with its Bolt replay claim, and the board server's HTTP routes.

Departures, all `PATTERN-test-only-code`, each admitting production code before a `main` package reaches it because its caller lands in a later commit:

- `board: render board-data.js, the board's view projection (fix)`: two production functions.
- `board: refuse a write whose entry revision is stale`: `Board.Expecting`.
- `board: embed the viewer page and claim its files in the replay`: `ViewerPage`.

## 05-board-server-lifecycle

Built the board server's lifecycle and the `lym board serve` and `lym board open` verbs.

No departures listed.

## 07-vscode-tasks-and-keys

Built the `board open` worktree task and its keys, fabric's workspace-content seam,
and the ide hook that writes `beam start` into the hub and beam workspace files.

Departure, `PATTERN-test-only-code`:

- `fabric: a workspace-content seam for the workspace files`: `InstallWorkspaceContent` is admitted before a `main` package reaches it, because the root command installs its hook in a later commit.
