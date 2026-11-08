# Slack: self-switching machine presence, reed toggles, reply forwarding

Presence now belongs to the machine and changes when the operator acts.
A Slack message from the operator sets it to away, and a line they type in an orch's pane sets it back to terminal.
While the machine is away, lym forwards each orch's final reply to that orch's Slack channel.
reed's status bar gains two toggles, `slack` (connection) and `away` (presence).

## What changes

- **Presence and a connection switch per machine.**
  `slack-presence.json` and the new `slack-switch.json` sit beside the Slack credentials in the per-user lym directory (`hubgeom.SlackMachine`).
  So away holds for every hub and orch on the machine, and every reader (the dobby's wait reminders, the relay, the orch's resume pointer) follows through `Geometry.PresencePath`.
  While the switch is off, no relay start spawns a relay; the new outcome is `OutcomeSwitchedOff`.
  `lym slack start` and `away` switch it back on.
  `slackengine.Connected` reports whether a relay holds the machine's socket lock.
- **Notices come from the relays.**
  `SetPresence` is gone, and writers only call `WritePresence`, which skips an unchanged write.
  Each running relay reads the presence at its start and mails its own hub's orch and beam orchs a `[slack presence]` notice when it sees a change.
  So a change made by any route (CLI, Slack text, hook, toggle) gets the same notice.
- **Auto-away.**
  Before forwarding any operator message that is not `away`/`back`/`status`, the inbound read sets the presence to slack.
  A failed write is logged and never holds the message.
- **Auto-back and reply forwarding through a turn hook.**
  `shuttleengine.Spec.TurnHookCommand` is a new, provider-neutral field.
  The claude engine pipes the UserPromptSubmit input and the turn-end Stop input to it, with output discarded and the exit statuses unchanged.
  In the merged creel Stop command it runs only on the path that records a turn end, so a Stop that handed out mail runs no turn hook.
  `HookInput` gains `Prompt` (via `typedInstruction`, so slash commands and task notifications come back empty) and `Reply` (`last_assistant_message`).
  The orch spec sets the hook to `lym slack hook`.
  - On a prompt while away, the hook sets terminal unless the line is lym's own: empty, opening `[creel]` (`creelengine.LinePrefix`) or `[lym]`, or matching an inject creel journaled into this strand within two minutes (`LogEntry.TypesInject`).
  - On a main-thread turn end while away, it posts the reply (cut at 7000 bytes) to `<shortname>:SLACK` as the orch strand, so the relay posts it in that orch's channel.
    The creel claim limits both paths to a process in the orch's pane.
- **reed toggles.**
  Line 3 draws `slack` and `away` after `follow`, lit from the `#()` job `lym slack bar`, which prints `slack-on|off away-on|off`.
  The job's line lives in `@lym_slack_state_command`.
  `status-interval` is pinned to 5 s, so a change by any route shows within that interval.
  A click runs `lym slack toggle slack|away` from the session's run directory, then redraws the client through `reedengine.ShowToggleOutcome`, which also shows any refusal.
  - `toggle slack` switches the connection off and ends every live hub's relay, or switches it on and starts this hub's.
  - `toggle away` flips the presence.
- **Sender names carry the machine** (added after Discussion).
  Slack posts go under `agentname.FormatMachineAddress`, `<machine>::<sender>`, such as `neve::ly:orch`.
- **Orch role text.**
  `role-slack.md` drops the "post every operator-bound message to SLACK while away" duty that the hook now covers.
  It keeps one line on sending to Slack on purpose.

## Design decisions

- Auto-back reads the prompt in a UserPromptSubmit hook rather than in the orch watcher's event read.
  The same hook carries the Stop forwarding, so the reply goes out as the orch strand and lands in that orch's channel under its name.
- The toggles pull their state through a tmux `#()` job instead of having each changer push options to every reed server.
  So a crashed relay unlights `slack` without anyone writing it.
- "Connected" means a relay holds the machine's socket lock, which includes a holder that is dialing again.

## Bounds

- An older per-hub `presence.json` is not read: after deploy the machine starts at terminal.
- A hub whose relay is down gets no presence notice. The orchs no longer need one, since the hook reads the presence at each turn end.
- The codex engine installs no turn hook, so an orch on codex neither auto-switches nor forwards replies.
- Recognizing an inject relies on its journaled line equalling the submitted prompt within two minutes.
