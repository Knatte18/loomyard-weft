# reed: run the capability probe on a private tmux socket

Two concurrent reed boots on one hub socket could fail with `run list-commands: exit status 1: server exited unexpectedly`.
That was the cause of the `TestBoot_TmuxConf/AppendingFileIsSourcedOncePerServerStart/TwoWorktreesBootedConcurrently` flake.

`list-commands` is answered by a server, so on a socket with none tmux starts a transient server that exits as soon as it answers.
The capability probe ran it on the hub's socket.
A sibling boot whose client connected to that socket while the transient server was exiting got `server exited unexpectedly`.
Twelve concurrent `tmux -f /dev/null -L <key> list-commands` loops on one key reproduced it: 1 failure in 2400 calls, with that message.

Closes #712

## Change

Off Windows, `probeCapabilityLocked` (`internal/reedengine/probe.go`) now runs both probe calls (`-V`, then `list-commands`) on a fresh `-L` key of its own, still with `-f /dev/null`.
No client of the hub's socket can reach the transient server any more, so the race is gone from production code rather than worked around in the test.
Once the server is gone, the probe removes its socket file with `removeSocketFileOnceGone`.
A removal failure is logged and never fails the probe.
The `reedengine` package doc (`doc.go`) describes the new behavior.

## Design decisions

- The key is `lyx-p` plus seven random hex digits, 12 bytes.
  A hub socket key that passes reed's path-length check is never shorter than 12 bytes (`lyx-<hash8>`), so the probe's socket path fits wherever the hub's does.
  Two concurrent probes drawing the same key would share one transient server, which only costs the isolation.
- Windows keeps the engine's own socket, since psmux has no `/dev/null`.
- The socket directory comes from the existing `socketDirFromEnv`, as `removeStaleSocket` already uses it.
- Unchanged: the boot loop, the stale socket-holder reap, the tmux_conf claim, and the gate slot pool that `publish_verify` runs outside.
- The cost is one extra transient tmux server per `up` or `resume`.

## Tests

`TestProbeCapability_LeavesTheHubSocketAndNoProbeSocketBehind` (tmux tier, not parallel) probes a hub with no server.
It asserts that no server and no socket file stand on the hub's key, and that no `lyx-p*` socket file is left in the socket directory.
The existing probe unit test still pins exactly two calls, `-V` then `list-commands`.

## To check by hand

- `lyx gate test --tags tmux ./internal/reedengine` was not run for this description, so run it, in particular `TestBoot_TmuxConf`.
- A tmux whose transient server outlives its client for longer than the socket-gone wait leaves the probe's socket file behind in the per-user socket directory.
  This was not exercised.
