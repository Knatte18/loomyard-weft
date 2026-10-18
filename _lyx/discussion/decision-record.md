# Decision record — tmux-conf-concurrent-boot-flake

## Goal

Two reed boots on one hub socket at once never fail with `run list-commands: exit status 1: server exited unexpectedly`.
`TestBoot_TmuxConf/AppendingFileIsSourcedOncePerServerStart/TwoWorktreesBootedConcurrently` stops flaking because the race behind it is gone from production code, not because the test works around it.

## Scope

- In: the capability probe in `internal/reedengine/probe.go`, its doc in `internal/reedengine/doc.go`, and the tests that pin the probe's socket.
- Out: the boot loop, the stale socket-holder reap, the tmux_conf claim and the gate slot pool that `publish_verify` runs outside.

## Decisions

### The probe's list-commands runs on a private socket off Windows

Decision: off Windows, the capability probe's two invocations address a fresh `-L` key of their own instead of the hub's, still with `-f /dev/null`, and the probe removes that key's socket file after it answers.
Windows keeps the hub's socket, as psmux has no `/dev/null`.

Rationale: `list-commands` is answered by a server, so on a socket with no server it starts a transient one that exits as soon as it answers.
A concurrent boot's tmux client that connects to the hub socket while that transient server exits gets `server exited unexpectedly`.
Reproduced with twelve concurrent `tmux -f /dev/null -L <key> list-commands` loops on one key: 1 failure in 2400 calls, with that exact message.
A private key takes the transient server off the hub socket entirely, so no hub-socket client can reach it.

### The private key is `lyx-p` and seven random hex digits

Decision: the key is 12 bytes, drawn per probe from a random source.

Rationale: a socket key that passes reed's path-length check is never shorter than 12 bytes (`lyx-<hash8>`), so the probe key fits wherever the hub's does.
A collision between two concurrent probes only restores the shared transient server of today.

## Constraints

- `PATTERN-lyx-temp-root` is unaffected: the probe socket lives in tmux's own per-user socket directory, as every reed `-L` socket does, and is removed by its literal key.
- `PATTERN-told-geometry` holds: the socket directory comes from the existing `socketDirFromEnv`, as `removeStaleSocket` already uses it.
- The socket-file removal is best-effort: a failure is logged and never fails the probe.

## Auto-mode assumptions

- The flake's cause is the probe's transient server on the hub socket; the boot loop's `new-session` and `has-session` race only against a dying server, and the probe's transient server is the only one a concurrent boot creates there.
- The cost of one extra transient tmux server per `up` or `resume` is negligible.

## Open risks

- A tmux whose transient server outlives its client for more than the socket-gone wait leaves the probe's socket file behind in the per-user socket directory.

## Acceptance criteria

- Off Windows, a probe against a hub with no server leaves no server and no socket file on the hub's key, and no probe socket file behind.
- The probe still issues exactly `-V` then `list-commands`.
- `TestBoot_TmuxConf` passes on the tmux tier.
