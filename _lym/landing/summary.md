# Codex sessions read idle, end on exit, and reach creel

Fixes the three Codex engine faults the pane/load trial found (`docs/research/codex-engine-trial-pane-load-2026-11-03.md`).

## Idle probe of an opened session

`Runner.SessionIdle` resolved the engine with an empty `RunState` for a strand with no run record, so it took the hub default engine.
A Codex session from `lym shuttle open` was read with the Claude reading and answered `NoInputBox` with the composer on screen, so creel never typed its mail.
It now takes the engine from the live strand's provider, as `Runner.Send` does.
A guid that has no run record and is not a live strand now fails instead of reading through the default engine.

## A Codex run reads ended after Codex exits

Codex fires no hook on exit, so a run's state stayed `asking/awaiting-input` after `/exit` while the loop recorded `died`.
The launch line now appends an exit record, `{"hook_event_name":"LymExit"}`, to the run's events file once Codex exits, before the pane's shell exits.
`codexengine` parses it as a process-ending session end with reason `codex-exit`, so the state reads dead as a Claude run does.
The record has no stamp line, so it carries no time.
An opened session has no events file and gets no exit record.
This adds `Shell.AppendLine` to `internal/shell`, with a POSIX and a pwsh form; the POSIX form uses `printf '%s\n'` so the line is never read as a format.

## Creel reachable from the Codex sandbox

The launch line, for runs and opened sessions, now sets `sandbox_workspace_write.network_access=true`.
Codex's Linux sandbox refuses every socket connect without it, a Unix socket's included, so `lym creel read` and `post --reply` could not reach the daemon.

Design decisions:

- Codex has no setting that opens Unix sockets alone, and its network proxy forwards a Unix socket only for HTTP requests, which creel's protocol is not.
  Network access is the narrowest opening available, so a session's shell commands now have the network a Claude seat's have.
- Writes stay bound to the workspace, the writable roots and the temporary folders.
- The Bolt `.lym` stays read-only, so a client in the session that finds no daemon still cannot start one.
  The daemon that delivers the session's mail is already up.
- The `codexengine` package doc states what is opened and why.

## To check by hand

Only the unit tier ran, with no regression check and no live Codex.
The trial's end-to-end case 6 runs after this lands.

- In a real Codex run on `-a never -s workspace-write`, `lym creel read` and `lym creel post --reply <id>` reach the daemon with `network_access=true`.
- `lym creel post` to a session from `lym shuttle open --model luna[medium]` is typed within seconds, with no `no-input-box` stall in `lym creel log`.
- After `/exit` in a Codex run's pane, `lym shuttle state` reads ended and no `session state disagrees` line is logged.
