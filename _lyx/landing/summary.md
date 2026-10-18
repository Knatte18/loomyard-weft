# reed: run the capability probe on a private tmux socket

Fixes the `TestBoot_TmuxConf/AppendingFileIsSourcedOncePerServerStart/TwoWorktreesBootedConcurrently` flake (issue #712), where a boot failed with `run list-commands: exit status 1: server exited unexpectedly`.

## Cause

The capability probe runs `list-commands` before every boot.
`list-commands` is answered by a server, so on a hub socket with no server running tmux starts a transient one that exits as soon as it has answered.
The probe used the hub's own `-L` socket.
When two worktrees of one hub boot at once, one boot's tmux client could connect to the other boot's transient server while it was exiting, and failed with `server exited unexpectedly`.
The race is in production, not only in the test: two `lyx reed up` calls from sibling worktrees hit it the same way.
Twelve concurrent loops of `tmux -f /dev/null -L <key> list-commands` on one key reproduce it, about once in 2,400 calls.

## Change

- `probeCapabilityLocked` (`internal/reedengine/probe.go`): off Windows, the probe now copies the engine's `TmuxCmd` and points it at a fresh `-L` key per call.
  It keeps `-f /dev/null`, so an operator config cannot keep the transient server alive.
  After the probe it removes that key's socket file with the existing `removeSocketFileOnceGone`, and logs a failure without failing the probe.
  Windows keeps the hub's socket, as before.
- `newProbeSocketKey`: the key is `lyx-p` plus seven random hex digits.
- `internal/reedengine/doc.go`: the multiplexer-contract section describes the private probe socket.
- New tmux-tier test `TestProbeCapability_LeavesTheHubSocketAndNoProbeSocketBehind` (`serverconfig_integration_test.go`): after a probe on a hub with no server running, the hub's key has no server and no socket file, and the socket directory holds no probe socket.

## Design decisions

- **Private socket, not a retry.** Retrying `list-commands` would hide only the probe's own failure.
  The transient server would still sit on the hub socket, where a concurrent boot's `new-session`, `has-session` or `display-message` could reach it while it exits.
- **The key is 12 bytes.** The shortest hub key reed forms is the standalone `lyx-<hash8>`, which is also 12 bytes.
  So wherever the hub's socket path passes reed's path-length check, the probe's path fits too.
  The socket lives in tmux's per-user socket directory, like every reed `-L` socket, rather than under `lyxtemp`'s longer run-directory path.
- **The new test is not parallel.** It scans the socket directory that every test in the binary shares, and a concurrent probe would leave a socket there mid-scan.
- **Cost.** Every `up` or `resume` now starts one short-lived tmux server, including when the hub's server is already up.

## Bounds

- A collision between two concurrent probes' random keys brings back the old shared transient server for those two probes only.
- A transient server that outlives `socketGoneWait` (2s) leaves its socket file in the per-user socket directory.
