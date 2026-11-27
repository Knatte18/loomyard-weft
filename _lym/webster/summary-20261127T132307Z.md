# Board viewer, its server and the VS Code keys

All three batches are recorded `done`, covering the plan's nine cards.

## What was built

- **01-board-view-projection** (head `ea3ed3333`): the `board-data.js` view projection, the entry-revision check on writes, the embedded `board.html` page with its Bolt replay claim, and `boardserve`'s HTTP routes.
  It first stopped as `waiting`, because the test-only-code guard failed on functions with no caller until a later batch;
  the parent authorized temporary allowlist entries and the batch resumed.
- **05-board-server-lifecycle** (head `5fd0dd530`): the board server's lifecycle and the `lym board serve` and `lym board open` verbs.
- **07-vscode-tasks-and-keys** (head `68c4add1c`): the `board open` worktree task and its keys, fabric's workspace-content seam, and the `beam start` workspace task.

No batch reported a rename off the plan.

## Departures

- **01-board-view-projection**, all `PATTERN-test-only-code`:
  - card 1: two production functions admitted before a `main` package reaches them;
  - card 2: `Board.Expecting` admitted the same way;
  - card 3: `ViewerPage` admitted the same way.
- **05-board-server-lifecycle**: none.
- **07-vscode-tasks-and-keys**: card 8 admitted `InstallWorkspaceContent` the same way, until the root command installs its hook in a later commit.

## Scope guard

The scope guard found these files changed outside each batch's own targets.
The findings are informational.

- **01-board-view-projection**:
  - `cmd/lym/testonlycode_linux_test.go`
  - `internal/boardengine/expected.go`
  - `internal/boardengine/expected_test.go`
  - `internal/boardengine/view.go`
  - `internal/boardengine/view_test.go`
  - `internal/boardserve/server.go`
  - `internal/boardserve/write.go`
- **05-board-server-lifecycle**:
  - `cmd/lym/testonlycode_linux_test.go`
  - `internal/boardserve/run.go`
  - `internal/boardserve/state.go`
  - `internal/lock/lock.go`
  - `internal/lock/lock_test.go`
- **07-vscode-tasks-and-keys**:
  - `internal/fabricengine/hubworkspace.go`
  - `internal/ideengine/spawn_hub_integration_test.go`
  - `internal/vscode/config_test.go`
  - `internal/vscode/keybindings.go`
