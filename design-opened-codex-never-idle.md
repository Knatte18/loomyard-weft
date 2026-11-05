# An opened Codex session never reads idle, so creel types nothing into it

`opened-codex-never-idle` · Note · bug, shuttle, creel, codex

`Runner.SessionIdle` (internal/shuttleengine/session.go) resolves the engine with `engineFor(RunState{})` when the strand has no run record, which is the hub default engine (claude here).
A session from `lym shuttle open --model luna[medium]` has no run record, so its Codex pane is read with the Claude reading and the probe answers `NoInputBox` although the composer is on screen.

Seen 2026-11-05: `lym creel post` of a notice from EXTERNAL to the opened strand was never typed; `lym creel log` shows a `stalled` record, reason `no-input-box`, one second after the post.
The same notice to a Codex strand started by `lym shuttle run` was typed in full within seconds.
`Runner.Send` already picks the engine from the strand's provider tag for a recordless strand; the idle probe does not.

Found by the pane/load trial, docs/research/codex-engine-trial-pane-load-2026-11-03.md case 4.