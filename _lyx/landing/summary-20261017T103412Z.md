# Install no network gate in a git worktree outside a hub

Fixes #697.

`lyxcwd.Resolve` takes the parent of any git worktree root as the hub.
A `lyx` command run in a Claude Code agent worktree under `<prime>/.claude/worktrees/<name>` therefore resolved `<prime>/.claude/worktrees` as its hub,
and the root pre-run's `tendercli.InstallNetworkGate` installed the pause gate on that guessed hub.
The gate's in-flight network markers then created `<prime>/.claude/worktrees/_board/.lyx/pause/net` as untracked directories in the prime, which made `update-plugins.sh` refuse the dirty prime at a Land deploy.

`InstallNetworkGate` now installs nothing unless `preflight.BoardLyxPresent` reports the resolved hub's board-level `_lyx` present, the same predicate hub reconcile uses to tell a hub from a plain repository.
A process outside a hub was already held by no pause, since the record at a guessed path never existed, so the only change in behavior is that no marker is written there.

## Design decisions

- The fix refuses rather than resolving the real hub from the agent worktree's git common directory: that would widen `lyxcwd`'s geometry for every caller, while the bug is a write at a guessed path.
- The check sits in the gate installer, not in `lyxcwd.Resolve`, because standalone mode and plain-repository callers depend on `Resolve` succeeding outside a hub.

## Tests

`TestInstallNetworkGate_TellsThePairsSlugAndInstallsNothingOutsideAHub` now also creates a linked worktree at `<prime>/.claude/worktrees/agent`, installs the gate from it, and asserts the previously installed gate stays in place and no sibling `_board` appears.
Without the fix that case fails.
