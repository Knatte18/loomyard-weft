# Refuse or warn on a creel post to SLACK that the relay will not post

`lym creel post ly:SLACK "<text>"` answered `{"ok":true}` even when the mail could never reach Slack.
The relay drops mail from a strand that is no orch and from an orch whose slack button is off.
While the relay is down the mail waits unread, and the sender heard about neither case.

## What changes

A post from a strand or EXTERNAL to the hub's Slack relay program, `<shortname>:SLACK`, now asks the daemon's Slack bridge before anything is stored.

- **Refused, nothing stored:** a sender the relay would drop unposted.
  - A strand that is no orch: the way forward is to report the text in its pane or ask its orch to post it.
  - An orch with no slack button, such as a pair's: the way forward is to ask the hub's orch or a beam's orch.
  - An orch whose button is off: the way forward is to press the button on its status bar.
  - An orch thread counts as the orch of its worktree, matching how the relay routes its mail.
- **Stored, with relay state:** for every other sender the post answers `receiver_live` as whether the relay runs.
  A relay that is down adds `receiver_live: false` and a `creel: warning:` line naming `lym slack start`, because the mail waits until the relay reads it.

A post to SLACK from a program is unchanged and asks nothing.
lym's own program senders, such as the dobby reminders and the slack turn hook, keep their current behaviour.
A daemon started without the bridge is also unchanged.

## Design decisions

- **Refuse vs. `delivered: false`.** The entry allowed either.
  A post the relay will certainly drop is refused, so the sender reports in its pane instead.
  A post that only waits for a down relay is stored with a warning: refusing it would lose mail that a restarted relay still posts.
  The warning reuses the existing `receiver_live` and `warning` fields, so no new field was added.
- **Seam.** The check is a new `RelayPostReady` method on `creelengine.MachineBridge`, the existing seam through which the engine reads Slack state.
  `creelcli`'s `slackBridge` implements it from the forwarding record, the beam listing and `slackengine.RelayLive`.
  It reuses the button rule that `SendReady` already applies to mail across machines.

## Tests

- `creelengine` `TestStore_PostToTheSlackRelay` covers a post while the relay runs, a post while it is down (warning text pinned), a refused sender (nothing journaled), a program sender and a store without the bridge.
- `creelcli` `TestSlackBridge_RelayPostReady` covers live and down relays, an orch thread, a button that is off, a pair's orch and a non-orch strand, using a held relay lock and a real forwarding file.
- `creelfake.Bridge` gains `RelayPostReady` and `SetRelayLive`.
