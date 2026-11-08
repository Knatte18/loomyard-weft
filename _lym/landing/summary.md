# Slack relay: forward the operator's text without a `[slack]` body prefix

An operator message from Slack reached the orch as `[creel] #1033 from "ly:SLACK": [slack] OK. ...`:
the sender `<shortname>:SLACK` already names Slack as the source, and the body said it again.
The relay now posts the operator's text as it stands, followed by a line per attached file, with no marker of its own.

## Changes

- `internal/slackengine/inbound.go`: `forward` no longer prepends a prefix, and the `forwardPrefix` constant is gone.
  The 4096-byte creel cap now applies to the operator's text alone,
  so a message up to the full cap is delivered.
- `contracts/orchrole/role-slack.md`: the orch role's Slack file was the one reader that keyed on the prefix.
  It now lists the two notice kinds first, `[slack presence]` and `[slack relay]`,
  and reads any other mail from `<shortname>:SLACK` as the operator's own words, with pane authority.
- `internal/slackengine/doc.go` and `docs/overview.md` describe the forwarded mail without the prefix.
- The inbound and relay tests expect the bare text, and the creel-cap rows sit at 4096 and 4097 bytes.

## Design decisions

- **Drop the prefix everywhere rather than keep it where the sender does not show.**
  Creel's delivery line names the sender with each message, so the orch always sees where the mail came from.
- **Notices keep their prefixes and the operator's words take the rest.**
  Relay and presence notices are always written with their prefix,
  so no notice can read as operator words.
  An operator message that itself opens with `[slack relay]` or `[slack presence]` reads as a notice;
  that only lowers its authority and never raises it.
