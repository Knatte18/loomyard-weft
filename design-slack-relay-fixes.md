# Slack relay: skip its own bridge posts, and give relay commands a prefix

`slack-relay-fixes` · Task · bug, enhancement, slack · starting · Open-Window

## Part 1: the bridge poll skips its own posts

## Seen (operator, 2026-11-26, first live run between Neve and osl46)

Each relay reads its own bridge post as inbound mail and posts a false name-clash notice under it, on both machines:

- Neve's orch posted `[lym-bridge] from=neve::ly:orch to=osl46::norce:orch id=2650`;
  Neve's own relay (`neve::ly:SLACK`) then posted "a message names neve, a name this machine holds or held, as its sender; it was not delivered; way forward: ask the operator to give the other machine a name this machine never held".
- osl46's reply `[lym-bridge] from=osl46::norce:orch to=neve::ly:orch id=129 reply-to=2650` drew the same notice from `osl46::norce:SLACK`, naming osl46.

Delivery itself worked both ways: each mail reached the other orch, stamped `(other machine, trusted orch)`.

## Likely cause

`bridgeReader.handle` in `internal/slackengine/bridgein.go` skips its own post only when `message.User == r.botUserID`.
The relay posts with a custom `username` (`api.go`, the sender's `Username`, shown as `neve::ly:orch`).
Slack stores such a post as a `bot_message` carrying `bot_id` and `username` but no `user`, so the check never matches.
The post then reaches `handleEnvelope`, whose from-clash check fires.
The unit tests likely give the fake message a `User`, which is why they passed.

## Idea

Skip a message whose `bot_id` is this machine's own bot (from `auth.test`, beside the bot user id), and keep the `user` check as well.
A test feeds a message shaped as Slack returns a `username`-override post: `subtype: bot_message`, `bot_id`, no `user`.
Every outbound bridge post on both machines makes the false notice today, so it hides real clashes.

## Part 2: relay commands take a prefix

## Seen (operator, 2026-11-26)

The relay reads some of the operator's messages in an orch's channel as its own commands and never forwards them to the orch (`internal/slackengine/inbound.go` `handle`, `peercommand.go` `parsePeerCommand`):
`away`, `back` and `status` as the whole message, and any message of two or three words whose first word is `approve` or `revoke`.
So "approve it" or "approve plan B" to the orch becomes a peer command for a machine named `it` or `plan`.
The operator finds a bare reserved word odd.

## Idea

Give every relay command one prefix the operator would not write to an orch, such as `lym approve neve`, `lym status`, `lym away`,
and forward every other message unchanged.
A Slack slash command (`/lym approve neve`) is the other way, but it needs the app manifest changed and reinstalled on each machine.
The relay's answers, `docs/slack-app.md` and `contracts/orchrole/role-slack.md` name the new form.
