# Orch sessions turn on remote control themselves, switchable per hub

Every orch session lym launches or resumes now starts with Claude Code's remote control on, so the operator no longer types `/remote-control` after a context cycle or in a fresh orch.
A new `RC` button on the orch session's status bar turns it on in a session that runs without it.

## What changes

- **Launch.**
  `orch.yaml` gains `remote_control` (on in the template) and `remote_control_name`.
  While the switch is on, the hub orch and numbered orchs, fresh or resumed, get `--remote-control <name>` on the Claude launch line.
  The resume line a session rebuild replays carries it too.
  Writers, reviewers and task sessions get no flag, and a Codex orch gets nothing, since only an engine with the new `shuttleengine.RemoteController` capability has remote control.
- **Name.**
  `remote_control_name` is a template filled by `Config.RemoteControlSessionName` with `{machine}`, `{hub}`, `{role}` and `{name}`.
  The default `{machine}::{name}` is creel's cross-machine address, such as `neve::ly:orch`, so the phone shows the name creel uses.
  `LoadConfig` refuses a name holding a control character, and an empty name while the switch is on.
  A name that cannot be formed is logged at Warn and the session launches without remote control, so the switch never stops a bring-up.
- **RC button.**
  Reed's third status line draws `RC` between the slack and away toggles, lit green while remote control is on and dim while off.
  The button reads two hidden verbs, `lym orch rc-bar` for the state and `lym orch rc-on` for the click.
  A click queues creel's new `remote-control` command into the live orch strand, which types `/remote-control <name>` once the session is idle.
  The click uses the launch's name whatever the switch says, then records remote control on and redraws the clicking client's bar.
- **Creel.**
  `remote-control` joins the closed command set, and its argument is the session name, one line without control characters.
  `Runner.TurnOnRemoteControl` sends it through the engine's `RemoteControlSequence`.
- **Orch state.**
  `orchroute.FixedStrandName` is split out of `StrandOf`, so a launch can name the strand before reed lists it.

## Design decisions

- **The button shows what lym last set.**
  Claude Code gives lym no signal for remote control's state, so a launch with the flag or a click writes a per-strand record, `remote-control.json` in the orch state directory, and `rc-bar` reads it.
  A session whose remote control the operator turned off inside Claude Code still shows lit.
- **No record, no button.**
  `rc-bar` prints nothing when the record does not name the orch's live strand, as for an orch on an engine with no remote control, and reed draws no button then.
  A start that finds an orch strand already running without a record writes it off, so the button is drawn and a click can turn remote control on.
- **A click on a lit button sends nothing.**
  What Claude Code shows for a second `/remote-control` is unverified and may be a panel that blocks the orch pane and creel's mail into it.
  The refusal names `/remote-control` typed by hand for a stale record.
- **Other click refusals.**
  A worktree with no live orch strand, with several orch strands, or with an orch on an engine with no remote control refuses and sends nothing, each naming its way forward.
  A record write that fails after the command is queued leaves the click a success with a `warning` field.
- **Reed's button styling.**
  The RC button reuses the toggles' lit and unlit styles, with its own `#()` state job in `@lym_rc_state_command` and its click line in `@lym_rc_click_command`.
  Neither is pinned under a test binary, as with the Slack toggles.
- **`rc-bar` skips the orch wiring.**
  The bar runs it from every session, and the wiring refuses a task pair's worktree.
  The verb names live in `reedengine` once, and `orchcli` references them.

## To check by hand

Nothing here ran against a real Claude Code session.

- Start an orch with `remote_control: true` and confirm the session shows under the expected name on the phone.
- Start one with a name template holding `{hub}` and `{role}` and confirm the fill.
- Start an orch on a session without the flag, click `RC`, and confirm the command lands and the button turns green.
- Click `RC` on a lit button and confirm the refusal reaches the bar and nothing is typed.
- The already-running adopt path, where a start finds a live orch strand with no record, writes the off record; this branch is thinly tested.
- The verb names `rc-on` and `rc-bar` are declared once in `reedengine` and referenced from `orchcli`; confirm no literal copy remains in `reedengine`'s bar and binding strings.
- The summary text of the click for a Codex orch, and the `rc-on` help text, read as the code behaves: a Codex orch's click refuses and sends nothing.
