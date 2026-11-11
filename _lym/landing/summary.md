# Slack per orch: a button, machine-first channel names, no presence mail

The Slack relay used to make a channel for every orch of every hub on the machine, with no way to choose,
and named each one `<hub>-<machine>-<slug>`, the reverse of the creel address.
This change gives each orch its own `slack` button, renames new channels machine-first, and drops the `[slack presence]` notice.

## What changes

- **Per-orch button.**
  The prime's orch and each beam's orch have a `slack` button on reed's status bar; a pair's orch has none.
  An orch has no channel until its button is first pressed.
  The press makes the channel and switches that orch's forwarding on; a second press switches forwarding off and leaves the channel.
  The state is kept per orch in `forwarding.json` in the hub's `.lym/slack/` directory, so a restarted orch keeps its channel.
  An orch that retires loses its entry.
- **Relay follows the buttons.**
  The relay starts when a button goes on and ends when the last button of the hub goes off.
  The machine-wide connection switch (`slack-switch.json`, `slackengine` `switch.go`) is removed;
  `slackengine.Connected` remains and reads the socket lock.
- **`away` is unchanged.**
  It stays one machine-wide presence.
  The turn-end hook still posts a reply while the presence is `slack`, now only for an orch whose button is on.
  Mail posted to `SLACK` by an orch whose button is off is dropped.
- **Channel names.**
  A new channel is named `<machine>-<shortname>-<project handle>-<project slug>`, lower-cased,
  such as `neve-ly-p0-root`; the prime's orch takes the root project's name.
  A character Slack disallows becomes `-`, and an empty part is left out.
- **No presence notice.**
  The relay no longer mails `[slack presence]`, and its line is gone from `contracts/orchrole/role-slack.md`.
  `docs/overview.md` and `pattern/PATTERN-daemon-wakeups.md` follow.
- **`lym slack bar` and `toggle`.**
  `bar` takes `--dir` and prints a slack word only for a worktree whose orch has a button;
  `toggle slack` flips that orch's button, and refuses where the orch has none.

## Design decisions

- The channel map moved to `orch-channels.json`, and the old `channels.json` is never read.
  Existing channels are therefore neither renamed nor archived; the operator archives them in Slack.
  Only a channel made after this change takes the new name.
- The root project's handle and slug are derived inside `slackengine` and `slackcli` from what the board already exports.
  No new export was added to the board module, which the task's limits exclude.
- Changes outside the named packages are the wiring they need:
  `internal/hubgeom` (`ForwardingPath` replaces `SwitchPath`), `internal/reedengine` (the bar), docs, and a test in `internal/beamcli`.

## To check by hand

- Press the `slack` button on the prime's and on a beam's status bar in a live hub with login and setup done.
  Confirm each gets its own `<machine>-<shortname>-<handle>-<slug>` channel, a second press keeps the channel,
  and the relay ends once the last button is off.
- Restart an orch whose button is on and confirm it keeps its channel without a press.
- Confirm a pair's orch shows no `slack` button.
- No regression check ran for this task.
