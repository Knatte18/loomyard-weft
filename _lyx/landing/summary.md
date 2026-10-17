# Install no network gate in a git worktree outside a hub

`lyx` run from a Claude Code agent worktree under `<prime>/.claude/worktrees/<name>` no longer creates `<prime>/.claude/worktrees/_board/.lyx/pause/net`.
That untracked directory made `update-plugins.sh` refuse the dirty prime and failed a Land deploy.

`tendercli.InstallNetworkGate` resolved the hub through `lyxcwd.Resolve`, which takes the parent of any git worktree root as the hub.
In an agent worktree that parent is a guessed path, and the gate's in-flight markers were written under it.
The gate now installs only when `preflight.BoardLyxPresent` reports a board-level `_lyx` at the resolved location, the same predicate hub reconcile uses to tell a hub from a plain repository.

Closes #697

## Design decisions

- Installing no gate counts as the "refuse" the issue allows.
  A process outside a hub is held by no pause, as before: a record at a guessed path never existed, so it never held a call.
- The hub is still resolved only through `lyxcwd`, and the hub check reuses `preflight`'s predicate rather than a new path computation.
- Resolving the real hub from an agent worktree's git common directory is out of scope, as are other hub-state writers, which already refuse or gate on hub presence.
- Known limit: a hub whose `_board/_lyx` is damaged runs without the gate, as hub reconcile already skips it.

## Tests

`TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` now also covers a linked worktree at `<prime>/.claude/worktrees/agent`.
It asserts that no gate is installed and that no `_board` appears beside the worktree.
The existing assertions still cover a directory outside any git repository and a pair's code worktree, which still installs the gate with the pair's slug.

The `tendercli` package doc and the `InstallNetworkGate` comment describe the new boundary.

## To check by hand

Nothing beyond the tests.
