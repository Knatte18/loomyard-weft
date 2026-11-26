# Slack relay commands take a prefix, so an orch message that opens with approve or revoke is not swallowed

`slack-relay-command-prefix` · Note · enhancement, slack

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
