# Card 8 — bridge-records-and-calls

**Create:**
- `internal/slackengine#PeersFileName`
- `internal/slackengine#BridgeMapFileName`
- `internal/slackengine#BridgeNoticesFileName`
- `internal/slackengine#BridgePostsFileName`
- `internal/slackengine#Peer`
- `internal/slackengine#Peers`
- `internal/slackengine#ReadPeers`
- `internal/slackengine#UpdatePeers`
- `internal/slackengine#BridgeEntry`
- `internal/slackengine#BridgeMap`
- `internal/slackengine#ReadBridgeMap`
- `internal/slackengine#UpdateBridgeMap`
- `internal/slackengine#BridgeMap.ChannelOf`
- `internal/slackengine#BotUser`
- `internal/slackengine#API.BotUsersNamed`
- `internal/slackengine#API.MemberChannels`
- `internal/slackengine#API.PostTimestamped`
- `internal/slackengine#API.PostThreadReply`
- `internal/slackengine#API.UserAppID`
- `internal/slackengine_test#TestBridgeRecords`

**Edit:**
- `internal/slackengine#MachinePaths`
- `internal/slackengine#Geometry`
- `internal/slackengine#SlackClient`
- `internal/slackengine_test#fakeSlack`
- `internal/slackengine_test#newGeometry`
- `internal/slackengine_test#TestAPI`
- `internal/hubgeom#SlackMachine`
- `internal/hubgeom#SlackGeometry`
- `internal/hubgeom#TestSlackGeometry`
- `internal/hubgeom#TestSlackMachine`
- `cmd/lym#testOnlyCodeAllowed`

**Extends:**
- `internal/slackengine/doc.go`

**Uses:**
- `internal/hubgeom/doc.go`
- `internal/slackengine#recordLockPath`
- `internal/slackengine#API.call`
- `internal/slackengine#API.BotAppID`
- `internal/slackengine#ChannelEntry`
- `internal/slackengine#ChannelInfo`
- `internal/slackengine_test#fakeSlackAPI`

**Intent:**
The records and Slack calls the bridge stands on: D6's approved-peer record, D4's bridge map, the file names of D4's notice record and D7's post record, and the calls D6 and D8 make.

Files, all through `internal/state` and never refusing an unknown key (`PATTERN-tolerant-records`), named in the geometry `internal/hubgeom` builds (`PATTERN-lym-files`):
- `MachinePaths` and `Geometry` gain `PeersPath`, `slack-peers.json` in the credential directory beside `slack.json`, never in a config layer; `SlackMachine` and `SlackGeometry` fill it.
- `Geometry` gains `BridgeMapPath`, `BridgeNoticesPath` and `BridgePostsPath` in the hub's `.lym/slack/`; `SlackGeometry` fills them.
`Peers` holds `Machines map[string]Peer`, keyed by the machine name lower-cased; `Peer` holds `AppID`, `BotUserID` and `ApprovedAt`.
`ReadPeers` reads a missing file as no peers; `UpdatePeers` is the record's one writer, reading and writing under its lock and creating the directory private to the user, as `UpdateChannelMap` does.
`BridgeMap` holds `Channels map[string]BridgeEntry` keyed by channel id; `BridgeEntry` holds `Name`, `LocalOrch`, `Peer` (the peer's full address), `Operator`, `PeerInvited`, `Cursor` (the delivery cursor), `OperatorCursor`, `Threads`, `ThreadFloor`, `Renamed` and `Archived`.
`ReadBridgeMap` reads a missing file as an empty map; `UpdateBridgeMap` is its one writer; `ChannelOf` finds the open entry of a local orch and peer address.

Slack calls, each on `API` and added to `SlackClient` (an interface method takes no arrow):
- `BotAppID`, which `API` already has (`bots.info`), joins the interface.
- `BotUsersNamed` pages `users.list` and keeps the bot users that are not deleted whose `name` equals name, or every such bot user when name is empty (card 9's `approve <m> <app id>` lists every bot this way), then reads each one's app id with `UserAppID`, answering `BotUser{UserID, AppID}` in listing order.
- `UserAppID` reads a user's app id from `users.info`, the profile's `api_app_id`, empty for a human; card 11 reads a bridge channel's creator's app id with it.
- `MemberChannels` pages `users.conversations` for public channels, archived ones left out, answering each with its creator.
- `PostTimestamped` posts as `PostAs` does and answers the posted message's `ts`.
- `PostThreadReply` posts text in the thread with `thread_ts`, under the bot's default name.
`ChannelInfo` keeps its fields; `users.conversations` answers fill `Creator` as `conversations.list` does.
D11 holds: each call needs only a scope the manifest already holds (`users:read`, `channels:read`, `chat:write`).

`fakeSlack` implements the new methods from in-memory state: a bot directory (name, user id, app id), a bot-id-to-app map, its member channels, and a counter of timestamps for `PostTimestamped`, which logs the post as `PostAs` does; `newGeometry` fills the four new paths.
`TestAPI` (tier 0) gains rows over `fakeSlackAPI`: `BotUsersNamed` pages two `users.list` pages, skips a human, a deleted bot and a bot of another name, and reads each match's app id; with an empty name it keeps every live bot; `UserAppID` answers the profile's `api_app_id`; `MemberChannels` pages and answers the creator; `PostTimestamped` answers the `ts`; `PostThreadReply` sends `thread_ts`; a refusal of each is an `*APIError`.
`TestBridgeRecords` (tier 0) is table-driven over the two records: a missing file reads empty; an update keeps a key the record does not know; `ChannelOf` finds the open entry of a pair and skips an archived one.
`TestSlackGeometry` and `TestSlackMachine` gain the new paths in their want.
`internal/slackengine/doc.go`'s files paragraph gains the peers record, the bridge map, the notice record and the post record.
No main reaches the new functions yet, so `cmd/lym#testOnlyCodeAllowed` gains an entry for each, its reason naming the card that adds the first caller and removes the entry:
- `internal/slackengine.ReadPeers`, `internal/slackengine.UpdatePeers`, `internal/slackengine.API.BotUsersNamed` and `internal/slackengine.API.UserAppID`, with `no caller until card 9's approve and revoke; that card removes this entry`;
- `internal/slackengine.ReadBridgeMap`, `internal/slackengine.UpdateBridgeMap`, `internal/slackengine.BridgeMap.ChannelOf` and `internal/slackengine.API.PostTimestamped`, with `no caller until card 10's relay posts to bridge channels; that card removes this entry`;
- `internal/slackengine.API.MemberChannels` and `internal/slackengine.API.PostThreadReply`, with `no caller until card 11's relay reads bridge channels; that card removes this entry`.

**ImpactSummary:** `SlackClient` grows six methods, implemented by `*API` and the tests' `fakeSlack`, its only implementers; the geometry gains fields every builder fills through `hubgeom.SlackGeometry`, and named-field literals elsewhere compile unchanged.

**Commit:** 8: bridge records, peers record and the Slack calls they need
