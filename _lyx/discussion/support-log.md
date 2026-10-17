# Support log — lyx creates hub state under a Claude Code agent worktree

## Interview

A darn run holds no interview; the board entry and a reading of `lyxcwd`, `hubgeom` and `tendercli` are the inputs.

## Rejected alternatives

- Resolve the real hub from the agent worktree's git common directory: it widens `lyxcwd`'s geometry for every caller, while the entry only asks that no hub state be created at a guessed path.
- Refuse in `lyxcwd.Resolve` when the parent holds no `_board`: standalone mode and plain-repository callers depend on `Resolve` succeeding there.
- Guard `pauserecord.Network`'s marker write: the package is told its geometry and cannot tell a guessed record path from a real one.

## Review rounds

_No rounds yet._

## Question ledger

- Which writer created `<prime>/.claude/worktrees/_board/.lyx/pause/net`? Resolved: the network gate's in-flight markers, installed by the root pre-run of every `lyx` command.
- Resolve or refuse? Self-pick: refuse, by installing no gate outside a hub.
