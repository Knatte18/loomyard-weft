# Codex sessions read idle, end when Codex exits, and reach creel from the sandbox

`codex-trial3-faults` · Task · bug, codex, shuttle, creel · done

Three Codex engine faults the pane/load trial found (`docs/research/codex-engine-trial-pane-load-2026-11-03.md`). Fix all three; the trial's end-to-end case 6 runs after this lands.

## 1. An opened Codex session never reads idle (case 4)

`Runner.SessionIdle` (`internal/shuttleengine/session.go`) resolves the engine with `engineFor(RunState{})` when the strand has no run record, which is the hub default engine (claude here).
A session from `lym shuttle open --model luna[medium]` has no run record, so its Codex pane is read with the Claude reading and the probe answers `NoInputBox` although the composer is on screen.

Seen 2026-11-05: `lym creel post` of a notice from EXTERNAL to the opened strand was never typed; `lym creel log` shows a `stalled` record, reason `no-input-box`, one second after the post.
The same notice to a Codex strand started by `lym shuttle run` was typed in full within seconds.
`Runner.Send` already picks the engine from the strand's provider tag for a recordless strand; the idle probe does not.

Expected: the idle probe picks the engine the way `Runner.Send` does.

## 2. `lym shuttle state` keeps a Codex run asking after Codex exits (case 3)

An interactive Codex run held at its composer reads `asking/awaiting-input` in `lym shuttle state`.
After `/exit` in its pane the run ended `died`, and the shadow state model logged `shuttle: session state disagrees ... loop=died state=asking cause=awaiting-input` in the same second.
The state of a Codex session follows hook signals alone and Codex exit fires none, so nothing in the run files reads as dead.

Expected: a Codex run whose loop ended reads ended in `lym shuttle state`, as a Claude run does.

## 3. A Codex session cannot reach the creel daemon from its sandbox (case 4)

Inside a Codex run on `-a never -s workspace-write`, `lym creel read` and `lym creel post --reply <id>` both fail:

`creel: no daemon answers on /tmp/lym-1000/creel-ly-92073c9b.sock after starting one and 6 connects: dial unix ...: connect: operation not permitted; daemon log .../_bolt/.lym/creel/daemon.log: the daemon log could not be opened: ... read-only file system`

The daemon was up (it had just sent the same strand mail), so the sandbox refuses the AF_UNIX connect.
The Bolt `.lym/creel` directory is read-only there too, so the client's own attempt to start a daemon cannot write its log.
The codexengine doc says the session runs `lym creel read` and `post --reply` from the sandbox and reaches the daemon; it does not.
The reminder `[creel] N unread ... run lym creel read` is typed, but the strand cannot act on it, so agent mail to a Codex strand is unreachable.

Expected: a Codex strand reads and answers its mail. Whatever the route (sandbox settings that allow the socket, a writable root, or another path), it keeps the sandbox as tight as it can and the codexengine doc says what is opened and why.
