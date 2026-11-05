# Codex sessions read idle, end on Codex exit and reach creel from sandbox

Fixes the three Codex engine faults the pane and load trial found (`docs/research/codex-engine-trial-pane-load-2026-11-03.md`, B3, B4 and B5).

## An opened Codex session reads idle

`Runner.SessionIdle` resolved a strand with no run record to the engine set's default.
A session from `lym shuttle open --model luna[medium]` has no run record, so its Codex pane was read with the Claude reading, the probe answered `NoInputBox` beside a visible composer, and creel journaled `stalled no-input-box` instead of typing the notice.
The probe now takes the live strand's provider for a recordless strand, as `Runner.Send` and the interrupt path already do; a guid that is neither a run nor a live strand fails with the same refusal those paths give.

## A Codex run reads dead once Codex exits

Codex fires no hook on its own exit, and its liveness reads unproven, so `lym shuttle state` and the wait loop's shadow fold kept an exited run at `asking/awaiting-input`.
The launch line now appends an exit record, `{"hook_event_name":"LymExit"}`, to the run's events file after Codex exits and before the pane's shell exits.
`ParseSessionSignals` reads it as a session end with reason `codex-exit` that ends the process, so the fold settles `dead` as it does for a Claude run's `SessionEnd`, and the shadow's `died` pairing agrees.
The wait loop's process-death check also sees that session end, which gives the same `died` outcome the dead pane gives.
The record has no stamp line, so the dead state carries no time.
An opened session's line appends none, since it has no events file.

`internal/shell` gains `AppendLine` for this, in both dialects (`printf '%s\n' … >> …` and `Add-Content -LiteralPath … -Value …`), so the line stays shell-agnostic.

## A Codex session reaches the creel daemon

Both Codex launch lines now set `sandbox_workspace_write.network_access=true`.
Codex's Linux sandbox refuses every socket connect without it, a Unix socket's included; that refusal was the `connect: operation not permitted` the trial saw.
Running `socat` against a test Unix socket under `codex sandbox` (codex-cli 0.164.0) shows the plain `workspace-write` sandbox refusing the connect and the same sandbox with this setting allowing it.
Codex has no setting that opens Unix sockets alone; its network proxy forwards a Unix socket only for HTTP requests, which creel's protocol is not.
The opening is therefore network access for the session's shell commands, the same a Claude seat has, while writes stay bound to the workspace, the writable roots and the temporary folders.
The Bolt `.lym` stays read-only, so a client in the sandbox that finds no daemon still cannot start one; the daemon that delivers the session's mail is already up.
The codexengine package doc states what is opened and why.

## Tests

- `TestRunner_SessionIdle_RecordlessStrandReadsThroughProvider` pins the provider's engine reading a recordless strand, with the default engine never consulted.
- `TestParseSessionSignals` gains the exit record as a process-ending session end.
- `TestBuildLaunchCmd`, the Prepare tests and the skill-load test pin the network setting and the exit record on the launch line, and the network setting on an opened session's command.
- `TestShell_AppendLine` pins both dialects, with a `%` and a quote in the line.

## Design decisions

- The exit record lives on the launch line, not in a liveness prober: Codex keeps no session registry, and the files-only state reader needs a fact on disk.
- Network access over a writable Bolt `.lym/creel`: the connect, not the log write, is what failed while a daemon was up, and making Bolt state writable would widen writes rather than reach the daemon.
- End-to-end case 6 of the trial is left to run after this lands, as the entry says.
