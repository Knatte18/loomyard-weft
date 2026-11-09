# Slack per orch: a button, machine-first channel names, no presence notice

`slack-per-orch-activation` · Task · enhancement, slack, reed, creel · draft

## Seen (operator, 2026-11-09)

The Slack relay was started on OSL-1046 and made one channel for every orch of every hub on the machine, seven in all.
There is no way to choose which orchs get a channel; `slack.yaml` holds only `remind_after_min`.
The channel names read `<hub>-<machine>-<slug>`, the opposite order of the creel address `<machine>::<hub>:<slug>:orch`.
Each change of presence mails a `[slack presence]` notice to every hub's orch and every beam's orch, and wakes the idle ones; `contracts/orchrole/role-slack.md` says an orch need not track the presence, since the turn-end hook posts its reply.

## This task

- `away` stays one machine-wide setting: the operator away from the machine is away for every orch.
- Each orch that has a status bar, the prime's and each beam's, has its own `slack` button.
  An orch gets no channel until its button is pressed once; the press makes the channel and switches that orch's forwarding on, and a second press switches it off and leaves the channel in place.
  A pair's orch gets no button.
- The button's state is kept per orch in a file in the hub's `.lym/slack/` directory, beside the channel map and the presence files; the status bar reads it from there.
  It survives the orch's restart, so a restarted orch keeps its channel without a new press.
- The machine's Slack connection follows the buttons: the relay runs only while at least one orch is active.
  The `slack` toggle of the status bar, which switches the machine's connection today, becomes the per-orch button.
- The `[slack presence]` notice is removed: the relay no longer mails it, and its line leaves `contracts/orchrole/role-slack.md`.
- Channel names put the machine first, as the creel address does: `<machine>-<hub>-<slug>-orch`, with the machine name the one `machine-name-config` provides.
  Slack allows no colon in a channel name, so the hyphen stands for the creel address's separators.
- The channels that already exist keep their old names and are left alone: the relay does not rename or archive them, and the operator archives them in Slack.
  Only a channel made after this task takes the new name.

## Limits

- Only `internal/slackengine`, `internal/slackcli`, the status bar's Slack toggles in reed, the orch role file and the tests that pin them change.
- The turn-end hook's posting of a reply while the presence is `slack` is unchanged, but it posts only for an orch whose button is on.
