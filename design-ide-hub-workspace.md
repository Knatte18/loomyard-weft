The operator wants to watch the board (and running tasks) in VS Code while talking to the hub orchestrator in the prime.
Direction: `lyx ide spawn` for the prime generates `<hub>/_launchers/<prime>.code-workspace` with folders prime, `_board` and `_portals`, and opens that instead of the bare folder.
Lyx-owned, outside every worktree, so agents never see it; task windows stay one pair each.
`code --add` works today but is ad hoc.