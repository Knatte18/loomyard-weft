# Slack relay: skip its own bridge posts; prefix relay commands with lym

## Own bridge posts are no longer read as inbound mail

The bridge read skipped this machine's own post only when the post's `user` matched the bot's user id.
A bridge post goes out under the sender's `username` (`neve::ly:orch`),
and Slack stores such a post as a `bot_message` with a `bot_id` and no `user`.
Each relay therefore read its own outbound post as mail from another machine.
The post's `from` named this machine,
so the relay posted a false "a message names neve, a name this machine holds or held, as its sender" notice
under every outbound bridge post, on both machines.
A real name clash would have been lost among these notices.

`channelKeeper` now keeps the whole `auth.test` identity, not only the user id (`botIdentity` replaces `botUser`).
`isOwnPost` matches a message by its user id or its bot id, and is used in two places:

- `bridgeReader.handle`, so the read skips its own posts;
- `bridgeReader.authored`, so a resync counts this bot's own envelopes when it adopts a channel this bot made.
  A channel whose only envelopes were username-override posts used to be left unmapped.

Tests feed messages with a bot id and no user, for both the skipped delivery and the resync adoption.
Another resync row keeps the user id match covered.

## Relay commands open with `lym`

The relay used to take a bare `away`, `back` or `status`,
and any two- or three-word message opening with `approve` or `revoke`, as its own command.
None of these reached the orch, so "approve it" became a peer command for a machine named `it`.
Every relay command now opens with `lym`:

- `lym away`, `lym back` and `lym status`;
- `lym approve <machine> [app id]` and `lym revoke <machine>`.

The prefix and the verb match case-insensitively.
Every other message is forwarded unchanged, including a bare `status`, `approve it`, `lym back now`
and `lym` followed by a word the relay does not know, such as `lym board list`.
In a bridge channel, only `lym approve` and `lym revoke` are carried out,
and any other operator text there gets the existing "not delivered" answer.

The new form is named by the relay's answers and notices, the outbound hold line for an unapproved peer,
creel's way forward for a `peer-unapproved` return, `internal/slackengine/doc.go`, `docs/overview.md`,
`docs/slack-app.md` (with a new "Relay commands" section) and `contracts/orchrole/role-slack.md`,
which now tells the orch that relay commands never reach it.

## Design decisions

- The prefix is a plain word, not a Slack slash command.
  A slash command would need the app manifest changed and the app reinstalled on each machine.
- A message that opens with `lym` but is not a known relay command is forwarded as written,
  not answered with a usage line, so the orch gets everything that is not a relay command.

## To check by hand

- The overview still naming the bare relay commands, in the slack entry of `docs/overview.md`.
  The sentence now names the `lym` forms; confirm it matches the relay's behavior.
- No live Slack run of this change is recorded.
  With two paired machines, send a bridge message each way and confirm that neither relay posts the name-clash notice.
  In an orch channel, confirm that `lym status` is answered by the relay and a bare `status` reaches the orch.

## Merge danger

**Door:** two-way
**Blast radius:** slack
