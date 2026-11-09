# Slack per orch: a button, machine-first channel names, no presence notice

`slack-per-orch-activation` · Task · enhancement, slack, reed, creel · draft

Depends on: [`machine-name-config`](design-machine-name-config.md)

## Seen (operator, 2026-11-09)

The Slack relay was started on OSL-1046 and made one channel for every orch of every hub on the machine, seven in all.
There is no way to choose which orchs get a channel; `slack.yaml` holds only `remind_after_min`.
The channel names read `<hub>-<machine>-<slug>`, the opposite order of the creel address `<machine>::<hub>:<slug>:orch`.
Each change of presence mails a `[slack presence]` notice to every hub's orch and every beam's orch, and wakes the idle ones; `contracts/orchrole/role-slack.md` says an orch need not track the presence, since the turn-end hook posts its reply.

## Decision (operator, 2026-11-09)

- `away` stays one machine-wide setting: the operator away from the machine is away for every orch.
- Each orch has its own `slack` button in its status bar.
  An orch gets no channel until its button is pressed once; the press makes the channel and switches that orch's forwarding on, and a second press switches it off and leaves the channel in place.
- The button's state is kept per orch and survives the orch's restart, so a restarted orch keeps its channel without a new press.
- The machine's Slack connection follows the buttons: the relay runs only while at least one orch is active.
  The `slack` toggle of the status bar, which switches the machine's connection today, becomes the per-orch button.
- The `[slack presence]` notice is removed: the relay no longer mails it, and its line leaves `contracts/orchrole/role-slack.md`.
- Channel names put the machine first, as the creel address does: `<machine>-<hub>-<slug>-orch`, with the machine name the one `machine-name-config` provides (`osl46`, `neve`).
  Slack allows no colon in a channel name, so the hyphen stands for the creel address's separators.

## Open for Discussion

What happens to the existing channels that carry the old naming: they are adopted by name today, so they would be left orphaned and need archiving, by lym or by the operator.
Where the per-orch state lives, and how the status bar reads it for each orch.
Whether a beam's orch and a pair's orch each get the button.
