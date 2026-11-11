# Slack per orch: a button each, machine-first names, no presence notice

The Slack relay used to make a channel for every orch of every hub on the machine as soon as it ran,
and reed's `slack` toggle switched the whole machine's connection.
Each orch with a status bar now has its own `slack` button instead,
and an orch gets a channel only once its button is pressed.

## What changes

- **Per-orch button.**
  The hub's `.lym/slack/forwarding.json` records whether each orch's button is on (`slackengine.ReadForwarding`, `ForwardingOn`, `WriteForwarding`).
  It survives the orch's restart.
  `lym slack toggle slack`, which a click on the button runs from the session's worktree,
  flips the button of the orch that runs there: the prime's orch or a beam's.
  A pair's or task's worktree has no button and the verb refuses it.
  Switching a button on needs login and setup and starts the hub's relay;
  switching it off leaves the channel in place and ends the hub's relay once no orch of the hub has its button on.
- **The relay follows the buttons.**
  `EnsureRelay` and `EnsureRelayWhileAway` start nothing while no orch of the hub has its button on (`OutcomeForwardingOff`),
  so the machine's Socket Mode connection runs only while some orch has its button on.
  The relay watches the forwarding file and, at each wake and at its start,
  makes the channel of each orch whose button is on and that has none; a beam's only while the hub lists the beam.
  The machine-wide connection switch (`slack-switch.json`, `ReadSwitchedOff`, `WriteSwitchedOff`) is removed.
- **Forwarding respects the button.**
  The keeper makes no channel for an orch whose button is off.
  The outbound read drops mail of such an orch.
  The inbound read answers an operator's message in such an orch's channel and forwards nothing.
  The turn-end hook posts a reply only while the presence is `slack` and the orch's button is on.
  `away` stays machine-wide; `lym slack away` sets it even with no button on, waiting for the relay's first read only when one runs.
- **Channel names.**
  A new channel is `<machine>-<shortname>-<project handle>-<project slug>`, lower-cased, such as `osl46-mo-p1-adopt-takeover`;
  the prime's orch takes the root project, `osl46-mo-p0-root`.
  The relay asks the hub for the project through a new `Hub.Project` seam, which `slackcli` answers from the board.
  A retiring channel is renamed from whatever name it carries, and the entry's new `renamed` flag stops a second rename.
- **No presence notice.**
  The relay no longer mails `[slack presence]` to the orchs, and the line leaves `contracts/orchrole/role-slack.md`.
- **Status bar.**
  `lym slack bar --dir <dir>` prints `slack-on` or `slack-off` for the button of the orch that runs in `dir`,
  and no slack word for any other directory, then the away word.
  reed passes the session's `@lym_button_dir` and expands the state option as a format, so each session's bar reads its own orch,
  and it draws the `slack` toggle only while the line holds a slack word.
- `lym slack status` reports `forwarding`, the orchs whose button is on, in place of `connection_off`.

## Design decisions

- **The old channels are left alone by never reading the old map.**
  The channel map moves to `orch-channels.json`.
  The relay never reads `channels.json`, so it never reads, posts to, renames or archives a channel made before this change;
  the operator archives those in Slack.
- **A retired beam's button is dropped** with its channel's archive, so a later beam of the same slug starts with its button off.
- **Mail of an orch whose button is off is dropped, not held.**
  Holding it would deliver a backlog at the next press; the creel log keeps it.
  Mail from a beam the hub no longer lists used to go to the prime's channel;
  it is now dropped, since the beam's button went with its retirement.
- **Two small edits outside the record's listed packages.**
  `internal/hubgeom` builds the engine's geometry, so it loses the switch path and gains the forwarding path.
  `internal/boardengine` gains `RootProjectHandle`, so the prime's channel names the root project's handle without hard-coding `P0`.
  `docs/overview.md`'s slack entry is updated to match.

## Testing

The tests of `internal/slackengine`, `internal/slackcli`, `internal/reedengine` and `internal/hubgeom` are updated to the per-orch button.
New cases pin the forwarding record, the new channel name, the keeper refusing a channel for an orch whose button is off,
the relay making a pressed orch's channel at its wake, the drop of an off orch's mail, the answer in an off orch's channel,
the toggle's on and off paths and its refusal in a pair's worktree, the bar's line per worktree, and the hidden slack toggle in reed's bar.
