# Slack toggles in reed; presence per machine and switching by itself

Reed's status bar gets two toggles beside `follow`, and Slack presence now follows the operator without a command.

## What changes

- **`slack` toggle:** lit while the machine's Slack relay is connected.
  A click starts or stops the relay, so Slack can go off without touching presence.
  While the machine is switched off, no start of a relay spawns one, so the prime dobby's tick does not revive it.
- **`away` toggle:** lit while presence is `slack`.
  A click switches presence either way.
- **Redraw:** the toggles read a `#()` job running the hidden `lym slack bar`, and the bar redraws every 5 seconds.
  A change by any route shows within that interval.
- **Presence is per machine.**
  The presence, switch and socket-lock files all live in the per-user credential directory (`hubgeom.SlackMachine`).
  Away holds for every hub and orch on the machine.
  `WritePresence` leaves an unchanged value unwritten, so a repeat wakes no relay.
- **The `[slack presence]` notice** is now sent by each running relay when it sees the presence file change, to its own hub's orchs.
  The writer no longer sends it, so a change by any route (toggle, hook, relay, `lym slack away|back|stop`) is announced once per hub.
- **An operator message on Slack** sets presence to `slack` before the relay forwards it.
- **A line the operator types in an orch's pane while away** sets presence back to `terminal`.
  The new hidden `lym slack hook` verb does it from the `UserPromptSubmit` event.
  It leaves presence alone for a slash command or a prompt the provider submitted (empty prompt),
  a line opening `[creel]` or `[lym]`,
  and a prompt matching an inject creel journaled for that pane within the two minutes before (`LogEntry.TypesInject`).
- **Final replies reach Slack while away.**
  The same hook, on Stop, posts the turn's final reply to the orch's channel through creel.
  A turn with no text sends nothing, and a reply over 7000 bytes is cut.
  Subagent events do nothing.
  `orchcli` sets the hook as the orch session's `TurnHookCommand`; `claudeengine` runs it after the existing event recording, with output discarded and the recording's exit status kept.
- **Shipped orch text:** `role-slack.md` keeps one plain line on sending to Slack through creel and drops what the hook now covers.
- **Sender names carry the machine.**
  Posts to Slack under a sender's name use `<machine>::<sender>`, such as `neve::ly:orch`, formed by `agentname.FormatMachineAddress`.

## Design decisions

- **Hook, not orch judgment.**
  Lym sets presence and forwards replies; the orch tracks nothing.
- **Telling lym's lines from the operator's** is done in the hook by prefix and by matching against creel's journaled injects.
  Mail a hook hands the session arrives as a `[creel]` line, so it never switches presence.
- **A test binary must never run a Slack line.**
  The state job and the toggle click are built from `executablePath()`.
  Under `go test` that is the test binary, which ignores `slack bar` and runs the whole suite again;
  each session that suite creates pins the same job, and an earlier version of this change fork-bombed the machine that way.
  Three guards prevent it:
  - `slackLineExecutable` refuses the running test binary, so the state job and the click are pinned only for a real `lym`.
  - `reedengine`'s `TestMain` exits 2 on any positional argument, and `testmain_guard_test.go` proves it.
  - The hook commands in `claudeengine` name `lym` through `PATH`, not an `executablePath` pin.
- **A machine with no presence record** reads as `terminal`; a hub's old per-hub record is no longer read.

## To check by hand

- Click `slack` and `away` in a live reed session and watch both toggles redraw, including after `lym slack away` from a shell.
- With presence `slack` in a real orch pane: type a line (presence returns to `terminal`), then run `/compact` and let a creel inject land (presence stays).
- With presence `slack`, finish a turn in an orch and confirm its final reply appears in that orch's channel under the `<machine>::` sender name.
- Run the integration tier for `reedengine`, `slackcli` and `claudeengine` in full; the work ran only targeted package tests.
