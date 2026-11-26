# Slack relay: skip its own bridge posts; prefix relay commands with lym

## Own bridge posts are no longer read as inbound mail

The bridge read skipped this machine's own post only when the post's `user` matched the bot's user id.
A bridge post goes out under the sender's `username` (`neve::ly:orch`), and Slack stores such a post as a `bot_message` with a `bot_id` and no `user`.
Each relay therefore read its own outbound post as mail from another machine.
The post's `from` named this machine, so the relay posted the false notice "a message names neve, a name this machine holds or held, as its sender" under every outbound bridge post, on both machines.
A real name clash would have been lost among these notices.

`channelKeeper` now keeps the whole `auth.test` identity, not just the user id (`botIdentity` replaces `botUser`).
`isOwnPost` matches a message by its user id or by its bot id.
Two places use it:
- `bridgeReader.handle`, so the read skips its own posts;
- `bridgeReader.authored`, so a resync counts this bot's own envelopes when it adopts a channel this bot made.

Before this change, a channel whose only envelopes were username-override posts was left unmapped.

Tests feed messages shaped like the ones Slack returns for a username-override post: subtype `bot_message`, the bot id `B1`, no user.
- A row in `TestBridgeReceive`'s skipped-messages case expects no delivery and no post. On the old code it posts the from-clash notice.
- A row in the resync case expects the channel to be adopted. On the old code it is left unmapped.

## Relay commands open with `lym`

The relay used to take a bare `away`, `back` or `status`, and any two- or three-word message opening with `approve` or `revoke`, as its own command.
None of these reached the orch, so "approve it" became a peer command for a machine named `it`.
Every relay command now opens with `lym`:
- `lym away`, `lym back` and `lym status`;
- `lym approve <machine> [app id]` and `lym revoke <machine>`.

The prefix and the verb match case-insensitively.
Every other message is forwarded unchanged.
That covers a bare `status`, `approve it`, and the prefix followed by a word the relay does not know, such as `lym board list`.

In a bridge channel, only `lym approve` and `lym revoke` are carried out.
Any other operator text there gets the existing "not delivered" answer.

These places now name the new form:
- the relay's answers and notices: wrong app, unapproved machine, unapproved channel creator, and a command naming this machine;
- the outbound hold line for an unapproved peer, and creel's way forward for a `peer-unapproved` return;
- `docs/slack-app.md`, which gains a short "Relay commands" section;
- `contracts/orchrole/role-slack.md`, which now tells the orch that relay commands never reach it.

`internal/slackengine/doc.go` states the rule.

Tests for the new behavior:
- `inbound_test.go` gains rows showing that a bare `status`, `approve it` and `lym board list` are forwarded to the orch.
- The presence and peer-command rows now use the prefixed form, including mixed case.

## Design decisions

- The prefix is a plain word, not a Slack slash command. A slash command would need the app manifest changed and the app reinstalled on each machine.
- A message that opens with `lym` but is not a command the relay knows is forwarded as written, not answered with a usage line. The orch then gets everything that is not a relay command.

## Merge danger

**Door:** two-way
**Blast radius:** slack
