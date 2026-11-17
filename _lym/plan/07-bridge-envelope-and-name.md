# Card 7 — bridge-envelope-and-name

**Create:**
- `plan:internal/slackengine#BridgeEnvelope` -> `type BridgeEnvelope struct`
- `plan:internal/slackengine#bridgeEnvelopePrefix` -> `const bridgeEnvelopePrefix = "[lym-bridge]"`
- `plan:internal/slackengine#FormatBridgeMessage` -> `func FormatBridgeMessage(envelope BridgeEnvelope, body string) string`
- `plan:internal/slackengine#ParseBridgeMessage` -> `func ParseBridgeMessage(text string) (BridgeEnvelope, string, bool)`
- `plan:internal/slackengine#bridgeChannelPrefix` -> `const bridgeChannelPrefix = "bridge-"`
- `plan:internal/slackengine#BridgeChannelName` -> `func BridgeChannelName(one, other agentname.MachineAddress) string`
- `plan:internal/slackengine_test#TestBridgeMessage` -> `func TestBridgeMessage(t *testing.T)`

**Edit:**
- `internal/slackengine_test#TestChannelName`

**Extends:**
- `internal/slackengine/doc.go`
- `pattern/PATTERN-sole-parsers.md`

**Uses:**
- `plan:internal/agentname#MachineAddress`
- `plan:internal/agentname#ParseMachineAddress`
- `internal/agentname#FormatMachineAddress`
- `internal/machinename#CheckName`
- `plan:internal/creelengine#MachineEnvelopeWidth`
- `internal/slackengine#slackNamePart`
- `internal/slackengine#fitName`

**Intent:**
D5's envelope with its one formatter and one parser, and D4's bridge channel name.

`BridgeEnvelope` holds `From` and `To` (`agentname.MachineAddress`), `ID` and `ReplyTo` (`int64`, 0 for mail).
`FormatBridgeMessage` writes `[lym-bridge] from=<machine>::<name> to=<machine>::<name> id=<id>`, with ` reply-to=<id>` when `ReplyTo` is set, a newline, and the body as given (the caller escapes it for Slack).
`ParseBridgeMessage` takes a message's text as Slack delivers it and answers the envelope, the body after the first line, and true; it answers false for a first line that is not exactly that shape: the prefix, `from=`, `to=` and `id=` in that order, an optional `reply-to=` last, each id a positive integer, and each address parsed by `agentname.ParseMachineAddress(address, nil, machinename.CheckName)`.
It judges no machine as this one's; the reader does (card 11).
It is the sole parser of the envelope (`PATTERN-sole-parsers`), whose background list gains `Bridge envelope: internal/slackengine`.

`BridgeChannelName` writes `bridge-<end>--<end>`: each end is `slackNamePart` of the machine, `-`, and the creel name with `:` turned to `-`; the two ends sorted, so both machines compute one name; the whole cut by `fitName` at 80 characters with its hash.
For example `osl-1046::ly:orch` and `neve::ly:orch` give `bridge-neve-ly-orch--osl-1046-ly-orch`.

`TestBridgeMessage` (tier 0) is table-driven: format and parse round-trip `from`, `to`, `id` and `reply-to`, and mail with no reply-to; a body of several lines comes back whole; first lines that do not parse answer false: plain text, a missing `id`, `id=0`, fields out of order, a `from` with no `::`, a `to` whose machine part fails the rule, a `from` whose name is a program name.
A row pins that the formatted first line's length never exceeds `creelengine.MachineEnvelopeWidth` of its sender and receiver, with a 63-character machine and the widest ids, so creel's line check covers the envelope (see `## Shared Decisions`).
`TestChannelName` gains bridge rows: the example above; the same name with the ends given in either order; a long pair cut to 80 characters ending in the hash, and two such pairs sharing a prefix staying apart.
`internal/slackengine/doc.go` gains a paragraph on bridge messages: their envelope, its sole parser, and the channel name.

**ImpactSummary:** New functions only, with one test row added to `TestChannelName`; nothing calls them until cards 10 and 11.

**Commit:** 7: Slack bridge envelope and channel name
