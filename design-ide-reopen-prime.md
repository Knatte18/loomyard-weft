# Reopen the prime VS Code window with one short command

`ide-reopen-prime` · Someday · feature

Seen 2026-10-03: a snap auto-refresh of VS Code (rev 257 → 267 at 12:44) left the old instance running; batten's next `code <pair>` call then started a second instance, which restored the prime as a bare folder without the hub workspace's title and colour, and the operator had to close windows by hand.
Reopening the prime correctly takes `lyx ide spawn <prime-name>`, whose folderOpen chain (`reed up`, `reed add --unless-name orch`, `reed attach`) reattaches to the orch without starting a second Claude; a plain `code` on the prime gets the stale mill-managed `.vscode/tasks.json` and attaches nothing.
`lyx ide spawn` needs the slug and fails from the hub root, so the operator had to cd into the prime.
Idea: with no argument it opens the worktree it is run in, and from the hub root or another hub directory outside a worktree it opens the prime.
