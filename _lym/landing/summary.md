# Slack relay: forward the operator's text without a [slack] prefix
A Slack message reached the orch as `[creel] #1033 from "ly:SLACK": [slack] OK. ...`.
The sender `<shortname>:SLACK` already names Slack as the source, so the `[slack] ` body prefix said it twice.

`internal/slackengine/inbound.go` no longer adds the prefix.
The mail body is now the operator's text as they wrote it, followed by the `(file not relayed: …)` lines for any attached files.
`[slack presence]` and `[slack relay]` notices keep their prefixes, since they are not the operator's words.

## Design decisions

- The orch role text in `contracts/orchrole/role-slack.md` told the orch to treat mail opening `[slack]` as the operator's own words.
  That is the only reader that keyed on the prefix.
  It now reads the other way round: `[slack presence]` and `[slack relay]` mail is a notice that is not answered,
  and any other mail from `<shortname>:SLACK` carries the operator's words with the authority of a typed line.
- Without the prefix, a message with no text and no files would be posted as an empty body.
  `forward` now skips it, logs at debug level, and advances the cursor as for any handled message.
- The creel body cap of 4096 bytes now applies to the operator's text alone, with no 8-byte prefix counted against it.
  The cap tests pin 4096 as accepted and 4097 as refused.
- `docs/overview.md` and the `internal/slackengine/doc.go` package comment describe the unprefixed body.

## To check by hand

- A live orch pane that was started before this change carries the old role text and may still wait for `[slack]` before it answers.
  Confirm a running orch answers an unprefixed Slack message after its role text is reloaded.
