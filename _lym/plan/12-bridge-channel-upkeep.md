# Card 12 — bridge-channel-upkeep

**Edit:**
- `internal/slackengine#channelKeeper.inviteOperators`
- `internal/slackengine#channelKeeper.retire`
- `internal/slackengine_test#TestChannelKeeper`

**Extends:**
- `internal/slackengine/doc.go`

**Uses:**
- `internal/slackengine#ReadBridgeMap`
- `internal/slackengine#UpdateBridgeMap`
- `internal/slackengine#channelKeeper.renameDated`
- `internal/slackengine#ArchivedName`
- `internal/slackengine#ReadSetup`
- `internal/slackengine_test#keeperHarness`

**Intent:**
D4's upkeep of bridge channels: the operator's invite on a change, and the archive when an orch retires.

`inviteOperators` also invites the setup's operator into every open bridge-map entry whose `Operator` is another or none, and records it once the invite stands.
A refused invite mails one notice per episode, as for an orch channel, and joins the returned errors.
`retire` also archives the bridge channels of each beam orch it retires: for each open bridge entry whose `LocalOrch` is that orch, it renames the channel to its dated name (`renameDated`, `ArchivedName`) unless the entry is `Renamed`, archives it, and marks the entry archived.
A refused rename or archive leaves the entry open and marks the orch refused, as for its own channel; the prime's orch is never retired.
`retire`'s loop also visits each orch that still holds an open bridge entry and whose beam the beams read does not list, even when its own channel is archived and its button dropped, so a refused bridge archive is tried again at a later retire.

`TestChannelKeeper` (tier 0) gains rows:
- an operator change invites the new operator into every bridge entry that records another operator or none, and records it;
- a retired beam's orch has its bridge channels renamed under the dated name and archived, and its entries marked archived;
- a bridge entry of another orch is left open;
- a bridge archive Slack refuses leaves the entry open and marks the orch refused, and a later retire with retry archives it although the orch's own channel already stood archived;
- a refused operator invite into a bridge channel mails one notice per episode and joins the returned errors.
`internal/slackengine/doc.go` gains the bridge channel's operator invite and its archive on retirement.

**ImpactSummary:** Two keeper methods gain a loop over the bridge map beside their orch-channel loop; the wake loop's calls to them are unchanged.

**Commit:** 12: bridge channels follow operator changes and orch retirement
