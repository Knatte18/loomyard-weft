# Card 10 — bridge-sending

**Create:**
- `plan:internal/slackengine#bridgePosts` -> `type bridgePosts struct`
- `plan:internal/slackengine#bridgeNotices` -> `type bridgeNotices struct`
- `plan:internal/slackengine#markBridgeNotice` -> `func markBridgeNotice(path, key string) (bool, error)`
- `plan:internal/slackengine#clearBridgeNotice` -> `func clearBridgeNotice(path, key string) error`
- `plan:internal/slackengine#peerUnapprovedReason` -> `const peerUnapprovedReason = "peer-unapproved"`
- `plan:internal/slackengine#buttonOffReason` -> `const buttonOffReason = "button-off"`
- `plan:internal/slackengine#bridgeNameTakenReason` -> `const bridgeNameTakenReason = "bridge-name-taken"`
- `plan:internal/slackengine#bridgeArchivedReason` -> `const bridgeArchivedReason = "bridge-archived"`
- `plan:internal/slackengine#bridgeSender` -> `type bridgeSender struct`
- `plan:internal/slackengine#newBridgeSender` -> `func newBridgeSender(geom Geometry, mailbox Mailbox, slack SlackClient, clock Clock, keeper *channelKeeper) *bridgeSender`
- `plan:internal/slackengine#bridgeSender.settleAccepted` -> `func (s *bridgeSender) settleAccepted() error`
- `plan:internal/slackengine#bridgeSender.send` -> `func (s *bridgeSender) send(ctx context.Context) (bool, error)`
- `plan:internal/slackengine#bridgeSender.sendOne` -> `func (s *bridgeSender) sendOne(ctx context.Context, message creelengine.MachineOutbound, posts *bridgePosts) (bool, error)`
- `plan:internal/slackengine#bridgeSender.hold` -> `func (s *bridgeSender) hold(ctx context.Context, message creelengine.MachineOutbound, reason, noticeKey, line string) error`
- `plan:internal/slackengine#channelKeeper.ensureBridge` -> `func (k *channelKeeper) ensureBridge(ctx context.Context, localOrch string, peer agentname.MachineAddress, pin Peer) (string, string, error)`
- `plan:internal/slackengine#bridgePollBase` -> `const bridgePollBase = 5 * time.Second`
- `plan:internal/slackengine#bridgePollCeiling` -> `const bridgePollCeiling = 60 * time.Second`
- `plan:internal/slackengine#bridgePoll` -> `type bridgePoll struct`
- `plan:internal/slackengine#newBridgePoll` -> `func newBridgePoll(clock Clock) *bridgePoll`
- `plan:internal/slackengine#bridgePoll.wake` -> `func (p *bridgePoll) wake()`
- `plan:internal/slackengine#bridgePoll.run` -> `func (p *bridgePoll) run(ctx context.Context, round func(ctx context.Context) bool)`
- `plan:internal/slackcli#programMailbox.TakeMachine` -> `func (m *programMailbox) TakeMachine() ([]creelengine.MachineOutbound, error)`
- `plan:internal/slackcli#programMailbox.ReportMachine` -> `func (m *programMailbox) ReportMachine(id int64, reason string) error`
- `plan:internal/slackcli#programMailbox.SettleMachineDelivered` -> `func (m *programMailbox) SettleMachineDelivered(id int64, slackTS string) error`
- `plan:internal/slackcli#programMailbox.SettleMachineReturned` -> `func (m *programMailbox) SettleMachineReturned(id int64, reason string) error`
- `plan:internal/slackcli#programMailbox.DeliverMachine` -> `func (m *programMailbox) DeliverMachine(mail creelengine.MachineMail) (int64, error)`
- `plan:internal/slackengine#NewBridgeSender` -> `func NewBridgeSender(geom Geometry, mailbox Mailbox, slack SlackClient, clock Clock, keeper *ChannelKeeper) *BridgeSender`
- `plan:internal/slackengine#BridgeSender` -> `type BridgeSender = bridgeSender`
- `plan:internal/slackengine#BridgeSender.Send` -> `func (s *BridgeSender) Send(ctx context.Context) (bool, error)`
- `plan:internal/slackengine#BridgeSender.SettleAccepted` -> `func (s *BridgeSender) SettleAccepted() error`
- `plan:internal/slackengine#NewBridgePoll` -> `func NewBridgePoll(clock Clock) *BridgePoll`
- `plan:internal/slackengine#BridgePoll` -> `type BridgePoll = bridgePoll`
- `plan:internal/slackengine#BridgePoll.Run` -> `func (p *BridgePoll) Run(ctx context.Context, round func(ctx context.Context) bool)`
- `plan:internal/slackengine#BridgePoll.Wake` -> `func (p *BridgePoll) Wake()`
- `plan:internal/slackengine_test#TestBridgeSend` -> `func TestBridgeSend(t *testing.T)`

**Edit:**
- `internal/slackengine#Mailbox`
- `internal/slackengine#RunRelay`
- `internal/slackengine_test#fakeMailbox`
- `internal/slackengine_test#newRelayRun`
- `internal/slackengine_test#TestRunRelay`
- `cmd/lym#unboundedWaitFiles`
- `cmd/lym#testOnlyCodeAllowed`
- `internal/slackengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `internal/slackcli/doc.go`
- `internal/creelcli#GoClient.TakeMachine`
- `internal/creelcli#GoClient.ReportMachine`
- `internal/creelcli#GoClient.SettleMachineDelivered`
- `internal/creelcli#GoClient.SettleMachineReturned`
- `internal/creelcli#GoClient.DeliverMachine`
- `internal/creelengine#IsDaemonRefusal`
- `internal/creelengine#ErrDaemonRefusal`
- `internal/slackengine#FormatBridgeMessage`
- `internal/slackengine#BridgeChannelName`
- `internal/slackengine#ReadPeers`
- `internal/slackengine#UpdateBridgeMap`
- `internal/slackengine#ReadBridgeMap`
- `internal/slackengine#BridgeMap.ChannelOf`
- `internal/slackengine#ForwardingOn`
- `internal/slackengine#ReadSetup`
- `internal/slackengine#channelKeeper.ensure`
- `internal/slackengine#channelKeeper.botUser`
- `internal/slackengine#nameTakenCode`
- `internal/slackengine#isArchivedCode`
- `internal/slackengine#channelNotFoundCode`
- `internal/slackengine#refusalCap`
- `internal/slackengine#orchIcon`
- `internal/slackengine#slackEscape`
- `internal/slackengine#relayNoticePrefix`
- `internal/slackengine#wakeFloor`
- `internal/slackengine_test#manualClock`
- `internal/slackengine_test#fakeSlack`
- `internal/slackengine_test#newGeometry`

**Intent:**
D4's channel making, D7's sending and D10's bridge poll: the relay posts each machine-outbound message to its pair's bridge channel.
The sender goes in a new file `internal/slackengine/bridgeout.go` and the poll in `internal/slackengine/bridgepoll.go`, which `cmd/lym#unboundedWaitFiles` lists.
The tests are external (`slackengine_test`), so `export_test.go` gains `BridgeSender`, `BridgePoll`, their constructors, and `Send`, `SettleAccepted`, `Run` and `Wake`, as it exports the keeper and the outbound read.

`Mailbox` gains `TakeMachine`, `ReportMachine`, `SettleMachineDelivered`, `SettleMachineReturned` and `DeliverMachine`, the program-only verbs of cards 4 and 5 (interface methods, no arrow); `programMailbox` forwards each to its `GoClient`, and `fakeMailbox` scripts a take queue and records every report, settle and deliver.
`fakeMailbox` scripts a refusal creel answers as an error that wraps `creelengine.ErrDaemonRefusal`, so `IsDaemonRefusal` matches it.

This card is the first caller of `IsDaemonRefusal`, the `GoClient` methods `TakeMachine`, `ReportMachine`, `SettleMachineDelivered` and `SettleMachineReturned`, `FormatBridgeMessage`, `BridgeChannelName`, `ReadBridgeMap`, `UpdateBridgeMap`, `BridgeMap.ChannelOf` and `API.PostTimestamped`, so it removes their `cmd/lym#testOnlyCodeAllowed` entries of cards 4, 6, 7 and 8.
Nothing here calls `DeliverMachine` or `bridgePoll.wake`, so `cmd/lym#testOnlyCodeAllowed` gains `internal/slackcli.programMailbox.DeliverMachine` with the reason `no caller until card 11's relay delivers bridge mail; that card removes this entry`, and `internal/slackengine.bridgePoll.wake` with the reason `no caller until card 11's wake loop wakes the bridge poll; that card removes this entry`.

Records, both through `internal/state` under the geometry's paths:
- `bridgePosts` (`BridgePostsPath`) holds `Accepted map[int64]string`, the Slack timestamp of each message Slack accepted, and `Refusals map[int64]int`, apart from the outbox record of orch mail, so a confirmed orch-mail read never clears it.
- `bridgeNotices` (`BridgeNoticesPath`) holds `Marks map[string]time.Time`; `markBridgeNotice` sets a key and answers true only when it was unset, and `clearBridgeNotice` drops it, so a restarted relay posts no second notice for an episode.
  A send-side key is `<kind>:<local orch> <peer address>`, so it stands before the pair has a channel; dropping a channel from the bridge map leaves the marks.

`settleAccepted`, called once by `RunRelay` before the poll starts and so before any take, settles delivered every entry of `Accepted`.
It drops an entry once creel answers, a refusal included (`IsDaemonRefusal`; creel refuses an id it no longer holds), and keeps it on a lost connection for the next try.

`send` takes once per poll (`TakeMachine`), drops `Refusals` entries of ids take no longer returns, clears each send-side notice mark whose pair and kind no taken message is held for any more (so a hold creel ended by returning its message at the hour ends its episode, and the pair's next hold posts a notice), and calls `sendOne` per message oldest first; it answers whether it posted.
Since every poll takes afresh, a held message creel returned at its hour is never posted.
A taken id that `Accepted` already holds is never posted again: `send` settles it delivered with its recorded timestamp instead, dropping the entry once creel answers, so a settle lost to a dropped connection is made again at the next poll and not by a second post.
A report creel refuses for an id it no longer holds is logged and skipped; the next take decides.
`sendOne`, for a message from local orch O to peer P:
1. O's button off (`ForwardingOn`): report `button-off`. While no button is on the relay has stopped (`wakeLoop`), so creel returns the message at its hour.
2. P's machine has no pin (`ReadPeers`): hold `peer-unapproved`; one notice per episode, posted in O's channel and mailed to O as a `[slack relay]` notice, asks for `approve <machine>`.
3. No open bridge entry for (O, P): `ensureBridge`. It answers a channel id, or a hold reason with its notice key and line.
4. The entry's peer bot not invited: `Invite(channel, pin.BotUserID)`. A refusal holds the message with Slack's code and a notice telling the operator to invite `@lym-<machine>` by hand; `already_in_channel` counts as invited, so a hand invite clears the hold at the next poll; success sets `PeerInvited`.
5. Post `FormatBridgeMessage` (from `FormatMachineAddress(geom.Machine, O)`, to P, the id and reply-to, the body escaped) with `PostTimestamped` under `Sender{Username: <machine>::<O>, IconEmoji: orchIcon}`.
   Its timestamp is written to `Accepted` before the settle, so a restart never posts it again; then settle delivered, dropping the entry once creel answers.
6. A post Slack refuses with a code counts in `Refusals`; `is_archived` and `channel_not_found` also drop the entry from the bridge map, so the next send creates or finds it again.
   At `refusalCap` the message is settled returned with the code.
   A transport failure or a rate limit counts toward nothing and is tried at the next poll.
`hold` reports the reason through `ReportMachine` and, when `markBridgeNotice` answers true, posts the notice in O's channel (through the keeper's ensure, as orch mail is) and mails it to O as a `[slack relay]` notice; every send-side hold notice goes there, since the pair may have no channel yet.
Any hold ending, by a post or a changed reason, clears its key.
D4's holds count toward no refusal cap: they last until they clear or creel returns the message at its hour.

`ensureBridge` computes `BridgeChannelName(O on this machine, P)` and calls `CreateChannel`.
A new channel enters the bridge map with every cursor and the thread floor at the channel's start (empty), and `PeerInvited` false; the setup's operator is then invited and recorded, and a refused operator invite mails O one notice per episode naming the code and `lym slack setup --operator <user-id>`, as an orch channel's does, and is retried at each outbound wake by card 12's `inviteOperators`.
While a `bridge-name-taken` or `bridge-archived` hold of the pair lasts, it tries the create again, and so the `FindChannel` listing on `name_taken`, at most once per `bridgePollCeiling` per pair; between tries it answers the pair's last hold again.
- A channel found archived holds `bridge-archived`, with one notice naming the channel and the way forward: unarchive it in Slack, and the next try adopts it when this machine's bot or the approved peer's bot made it; for a channel anyone else made, also rename it to free the name, then archive it again if wanted. The relay never unarchives.
- A channel whose creator is this bot (`botUser`) or `pin.BotUserID` is joined when the bot is no member and adopted: delivery cursor at the channel's start, operator cursor and thread floor at its newest message (`Latest`).
- Any other creator holds `bridge-name-taken`, with one notice naming the channel and the way forward: rename that channel in Slack, which frees the name, and the next try creates the bridge channel.

`bridgePoll` runs `round` in a loop: the wait starts at `bridgePollBase`, doubles to `bridgePollCeiling` after a round that posted or delivered nothing new (a re-read held message is not new), and returns to the base after one that did.
`wake` ends the current wait at once, never within `wakeFloor` of the previous round's start.
`RunRelay` builds the sender, calls `settleAccepted`, and runs the poll with a round that calls `send` (card 11 adds the read).
`TestRunRelay`'s harness (`newRelayRun`) scripts an empty take queue, and its rows that key on the outbound read's five-second wait tell the bridge poll's waits apart, so the existing rows keep their meaning with the poll running.

`TestBridgeSend` (tier 0) drives a `bridgeSender` and a `bridgePoll` over `fakeSlack`, `fakeMailbox`, `newGeometry` and `manualClock`; rows:
- a message to an unapproved peer is not posted, reports `peer-unapproved`, posts one notice over three polls in O's channel and mailed to O, and is posted once the peer is pinned;
- a `peer-unapproved` hold whose message take stops returning (creel returned it) clears its mark, and the pair's next unapproved message posts a notice again;
- with O's button off and another orch's on, it reports `button-off` and is not posted;
- the first send creates `bridge-neve-ly-orch--osl-ly-orch`, invites the operator and the peer bot, records the entry, and posts the envelope;
- `name_taken` adopts a channel made by this bot, and one made by the peer bot;
- a channel made by anyone else holds `bridge-name-taken` with one notice naming the rename, and a lasting hold tries `CreateChannel` and lists with `FindChannel` once per ceiling as the clock advances;
- a refused operator invite into a new bridge channel mails O one notice naming the code;
- an archived channel holds `bridge-archived` with one notice naming the way forward, counted toward no cap over six polls; unarchived and made by either bot, the next try adopts it; unarchived and renamed when made by another, the next try creates the channel;
- a refused peer invite holds with Slack's code and a notice, is retried at each poll, and an invite that then stands clears the hold (`API.Invite` answers `already_in_channel` as a standing invite, which `TestAPI` already pins);
- a relay restarted during a `peer-unapproved`, `bridge-name-taken` or `bridge-archived` hold, with no channel, posts no second notice;
- an accepted message is not posted again after a restart, nor after a confirmed outbound read of orch mail, and the restart settles it delivered before any take; its entry stays while the settle fails on a lost connection and goes once creel answers, a refusal included;
- a settle lost on a dropped connection within one run: the next poll's take returns the message, and it is settled with its recorded timestamp, not posted a second time;
- a message Slack refuses five times, with an outbound read of orch mail between the refusals, is settled returned with the code; a transport failure and a rate limit between them count toward nothing; its `Refusals` entry goes once take no longer returns it;
- a post refused `is_archived`, and one refused `channel_not_found`, drops the entry, and the next send creates or finds the channel again;
- the bridge-post and bridge-notice records each keep a key they do not know across a write (`PATTERN-tolerant-records`);
- a message creel returned at its hour, which take no longer returns, is not posted when its hold clears;
- the poll's waits read 5, 10, 20, 40, 60 and 60 seconds while nothing is new, and 5 again after a post.
`pattern/PATTERN-daemon-wakeups.md`'s bullet of the Slack relay's loops gains the bridge poll, from five seconds doubling to sixty and never within a second of the previous round, and its final `and` moves to join the four loops.
`internal/slackengine/doc.go` gains the sending half, and its line `It runs the socket holder, the outbound read and a wake loop.` names the bridge poll too.

**ImpactSummary:** `Mailbox` grows five methods, implemented by `slackcli`'s `programMailbox` and the tests' `fakeMailbox`, its only implementers; `RunRelay` gains one goroutine and one call before it; orch mail's outbound read is untouched.

**Commit:** 10: the relay posts machine mail to bridge channels
