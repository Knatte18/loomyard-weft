# Card 8 — bridge-records-and-calls

**Create:**
- `plan:internal/slackengine#PeersFileName` -> `const PeersFileName = "slack-peers.json"`
- `plan:internal/slackengine#BridgeMapFileName` -> `const BridgeMapFileName = "bridge-channels.json"`
- `plan:internal/slackengine#BridgeNoticesFileName` -> `const BridgeNoticesFileName = "bridge-notices.json"`
- `plan:internal/slackengine#BridgePostsFileName` -> `const BridgePostsFileName = "bridge-posts.json"`
- `plan:internal/slackengine#Peer` -> `type Peer struct`
- `plan:internal/slackengine#Peers` -> `type Peers struct`
- `plan:internal/slackengine#ReadPeers` -> `func ReadPeers(path string) (Peers, error)`
- `plan:internal/slackengine#UpdatePeers` -> `func UpdatePeers(path string, mutate func(*Peers) error) error`
- `plan:internal/slackengine#BridgeEntry` -> `type BridgeEntry struct`
- `plan:internal/slackengine#BridgeMap` -> `type BridgeMap struct`
- `plan:internal/slackengine#ReadBridgeMap` -> `func ReadBridgeMap(path string) (BridgeMap, error)`
- `plan:internal/slackengine#UpdateBridgeMap` -> `func UpdateBridgeMap(path string, mutate func(*BridgeMap) error) error`
- `plan:internal/slackengine#BridgeMap.ChannelOf` -> `func (m BridgeMap) ChannelOf(localOrch, peer string) (string, bool)`
- `plan:internal/slackengine#BotUser` -> `type BotUser struct`
- `plan:internal/slackengine#API.BotUsersNamed` -> `func (a *API) BotUsersNamed(ctx context.Context, name string) ([]BotUser, error)`
- `plan:internal/slackengine#API.MemberChannels` -> `func (a *API) MemberChannels(ctx context.Context) ([]ChannelInfo, error)`
- `plan:internal/slackengine#API.PostTimestamped` -> `func (a *API) PostTimestamped(ctx context.Context, channel, text string, sender Sender) (string, error)`
- `plan:internal/slackengine#API.PostThreadReply` -> `func (a *API) PostThreadReply(ctx context.Context, channel, thread, text string) error`
- `plan:internal/slackengine#API.UserAppID` -> `func (a *API) UserAppID(ctx context.Context, userID string) (string, error)`
- `plan:internal/slackengine_test#TestBridgeRecords` -> `func TestBridgeRecords(t *testing.T)`

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

**ImpactSummary:** `SlackClient` grows six methods, implemented by `*API` and the tests' `fakeSlack`, its only implementers; the geometry gains fields every builder fills through `hubgeom.SlackGeometry`, and named-field literals elsewhere compile unchanged.

**Commit:** 8: bridge records, peers record and the Slack calls they need
