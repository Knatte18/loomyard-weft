# Card 11 — bridge-receiving

**Create:**
- `plan:internal/slackengine#bridgeResyncHint` -> `const bridgeResyncHint = "bridge-resync"`
- `plan:internal/slackengine#isBridgeChannelName` -> `func isBridgeChannelName(name string) bool`
- `plan:internal/slackengine#bridgeReader` -> `type bridgeReader struct`
- `plan:internal/slackengine#newBridgeReader` -> `func newBridgeReader(geom Geometry, mailbox Mailbox, slack SlackClient, hub Hub, liveHubs LiveHubs, clock Clock, keeper *channelKeeper, in *inbound) *bridgeReader`
- `plan:internal/slackengine#bridgeReader.read` -> `func (r *bridgeReader) read(ctx context.Context) (bool, error)`
- `plan:internal/slackengine#bridgeReader.readChannel` -> `func (r *bridgeReader) readChannel(ctx context.Context, channelID string) (bool, error)`
- `plan:internal/slackengine#bridgeReader.handle` -> `func (r *bridgeReader) handle(ctx context.Context, channelID string, entry BridgeEntry, message Message) (bool, bool, error)`
- `plan:internal/slackengine#bridgeReader.readThreads` -> `func (r *bridgeReader) readThreads(ctx context.Context, channelID string) (bool, error)`
- `plan:internal/slackengine#bridgeReader.operator` -> `func (r *bridgeReader) operator(ctx context.Context, channelID, thread string, entry BridgeEntry, message Message) error`
- `plan:internal/slackengine#bridgeReader.resync` -> `func (r *bridgeReader) resync(ctx context.Context) error`
- `plan:internal/slackengine#inbound.takeBridgeSignals` -> `func (in *inbound) takeBridgeSignals() (bool, bool)`
- `plan:internal/slackengine#BridgeReader` -> `type BridgeReader = bridgeReader`
- `plan:internal/slackengine#NewBridgeReader` -> `func NewBridgeReader(geom Geometry, mailbox Mailbox, slack SlackClient, hub Hub, liveHubs LiveHubs, clock Clock, keeper *ChannelKeeper, in *Inbound) *BridgeReader`
- `plan:internal/slackengine#BridgeReader.Read` -> `func (r *BridgeReader) Read(ctx context.Context) (bool, error)`
- `plan:internal/slackengine#BridgeReader.Resync` -> `func (r *BridgeReader) Resync(ctx context.Context) error`
- `plan:internal/slackengine_test#TestBridgeReceive` -> `func TestBridgeReceive(t *testing.T)`

**Edit:**
- `internal/slackengine#socketHolder`
- `internal/slackengine#socketHolder.route`
- `internal/slackengine#inbound`
- `internal/slackengine#inbound.takeHints`
- `internal/slackengine#readsForHint`
- `internal/slackengine#wakeLoop`
- `internal/slackengine#wakeLoop.wake`
- `internal/slackengine#RunRelay`
- `internal/slackengine_test#TestSocketHolder`
- `cmd/lym#testOnlyCodeAllowed`

**Extends:**
- `internal/slackengine/doc.go`

**Uses:**
- `plan:internal/slackengine#ParseBridgeMessage`
- `plan:internal/slackengine#ReadBridgeMap`
- `plan:internal/slackengine#UpdateBridgeMap`
- `plan:internal/slackengine#ReadPeers`
- `plan:internal/slackengine#markBridgeNotice`
- `plan:internal/slackengine#clearBridgeNotice`
- `plan:internal/slackengine#bridgePoll.wake`
- `plan:internal/slackengine#BridgePoll`
- `plan:internal/slackengine#NewBridgeSender`
- `plan:internal/slackengine#API.UserAppID`
- `plan:internal/slackengine#bridgePollBase`
- `plan:internal/slackengine#bridgePollCeiling`
- `plan:internal/slackengine#parsePeerCommand`
- `plan:internal/slackengine#inbound.runPeerCommand`
- `internal/creelengine#MachineMail`
- `internal/creelengine#MachineInboundRetention`
- `internal/creelengine#IsDaemonRefusal`
- `internal/creelengine#ErrDaemonRefusal`
- `internal/slackengine#retryWindow`
- `internal/slackengine#ReadChannelMap`
- `internal/slackengine#ForwardingOn`
- `internal/slackengine#inboundText`
- `internal/slackengine#channelKeeper.botUser`
- `internal/slackengine#channelKeeper.join`
- `internal/slackengine#ChannelEntry.PruneThreads`
- `internal/slackengine_test#holderRig`
- `internal/slackengine_test#manualClock`
- `internal/slackengine_test#fakeSlack`
- `internal/slackengine_test#fakeMailbox`

**Intent:**
D8: the relay reads its bridge channels, delivers bridge mail to creel, handles the operator in a bridge channel, and finds channels it lacks.
The reader goes in a new file `internal/slackengine/bridgein.go`; `export_test.go` gains `BridgeReader`, `NewBridgeReader`, `Read` and `Resync` for the external tests.
This card is the first caller of `GoClient.DeliverMachine`, `programMailbox.DeliverMachine`, `bridgePoll.wake`, `ParseBridgeMessage`, `API.MemberChannels` and `API.PostThreadReply`, so it removes their `cmd/lym#testOnlyCodeAllowed` entries of cards 6, 7, 8 and 10.

Routing.
`socketHolder.route` routes an event in a channel a hub's bridge map holds to that hub as it routes an orch channel's, the hint named by the channel (`hintName`).
An event in a channel no hub's orch map or bridge map holds writes `bridgeResyncHint` into the inbox of every live hub, at most once per `bridgePollBase` per hub (a `socketHolder` field of last hint times on its clock), and is acknowledged.
The existing `resyncHint` is unchanged, so a bridge resync never rereads every orch channel.
`inbound.takeHints` removes a hint naming a bridge-map channel, a thread hint `<channel>_<ts>` whose channel the bridge map holds, or `bridgeResyncHint`, and sets the matching signal on `inbound` (new fields) instead of an owed read; `readsForHint` therefore never sees a bridge channel's hint.
`takeBridgeSignals` answers and clears them; `wakeLoop.wake` calls the bridge poll's `wake` for a channel signal and marks a resync wanted for the other.
`RunRelay` builds the reader; the poll's round calls `send`, then the reader's `resync` when wanted, at the relay's start and once per `bridgePollCeiling`, then `read`.

Reading.
`read` reads each open channel of the bridge map after its delivery cursor (`History`) and its threads after each thread's cursor and the thread floor, as orch channels are read.
`handle` answers whether the delivery cursor moves past the message and whether it delivered; per message, in order:
1. This bot's own posts are skipped.
2. The operator's messages are handled once: only one after `OperatorCursor` is handled, then that cursor moves to it, so operator messages past a held peer message are handled and a reread handles none twice.
   `operator` runs `parsePeerCommand`: an `approve` or `revoke` naming this channel's peer machine runs `runPeerCommand` and answers in the channel; one naming this machine, its current or a former name matched case-insensitively as card 9 matches it, is left to the peer's relay.
   When `runPeerCommand` fails, `operator` returns its error and the cursor that holds the message does not move, so the read stops at the message and retries it at each poll.
   The reader keeps its own failing key, `<channel id> <ts>`, and that key's first failure time.
   Once the first failure is `retryWindow` old, it answers the failure where the command was written, with `PostThreadReply` for a thread reply, and moves the cursor that held the message past it: the operator cursor, or for a thread reply that thread's cursor.
   The bridge poll's goroutine runs the reader and the wake loop's goroutine runs `inbound`.
   `runPeerCommand` reads only `inbound` fields set when it is built and writes none (card 9);
   the reader posts its answers through its `SlackClient`, never through `inbound.answer`, so it touches none of `inbound`'s failing key, episodes or owed reads.
   Any other operator text is answered once, by the relay of the end that sorts first in the channel's name, as not delivered, with a pointer to the local orch's own channel.
3. Any other human's message, or one with no `bot_id`, is skipped.
4. A message whose first line `ParseBridgeMessage` refuses is skipped.
5. An envelope message posted longer ago than `MachineInboundRetention` is skipped.
6. An envelope whose `from` names this machine's current or a former name (`Hub.MachineNames`) is skipped with one notice per episode in the channel naming the clash and asking the operator to give the peer machine a name this machine never held.
7. The message's app is read with `BotAppID` and cached per bot id.
8. When the `from` machine has a pin and the app is not the pinned one, the message is skipped with one notice per episode naming the app, with the way forward: compare it with the other machine's `status`, and if it is the real peer, type `approve <machine> <app id>` here to pin it.
9. When the `from` machine has no pin, the message is held: the delivery cursor stays before it, and one notice per held episode asks for `approve <machine>` in this channel.
10. When the envelope's `to` is not this channel's local orch, the message is skipped.
11. When the local orch is no longer a live orch of this hub (the prime's, or a beam the hub's `Beams` lists), the message is skipped with one notice per episode naming the orch as retired.
12. A message to an orch whose button is off is held until the button is on.
13. Otherwise it goes to creel through `DeliverMachine` with sender, receiver, sending id, reply-to, the body as `inboundText` gives it, and its post time.
    Any refusal creel answers (`IsDaemonRefusal`), a reply-to naming an id this hub never sent included, is skipped with one notice per episode in the channel quoting creel's reason, so no refusal stops the channel; a lost connection holds the cursor for the next poll.
Read-side notice keys are `<kind>:<channel id>`, held whether or not the map holds the channel.
A read-side mark is cleared when its episode ends: when a read of the channel moves its delivery cursor past a message without that kind's state (a delivered message ends the unpinned-hold, wrong-app, from-clash, retired-orch and deliver-refusal episodes), so after `approve`, `revoke` and a new held message the new episode posts its notice.
A held message stops its channel's delivery cursor only, outside `retryOrGiveUp`'s five minutes; an approval or the button delivers held messages in order at the next read, moving the cursor past each, so a later `revoke` holds none of them again and posts no second notice.
An operator reply in a thread is handled as above and answered with `PostThreadReply` in its thread; every other thread reply is skipped.

Resync.
`resync` lists `MemberChannels` and reads each channel the bridge map lacks whose name `isBridgeChannelName` accepts (the `bridge-` prefix and a `--` between two ends, or for a name of 80 characters the prefix alone) and that no orch channel map of a hub of this machine holds, the hubs read through the `LiveHubs` seam, the relay's one listing of the machine's hubs, so on a machine named `bridge` no orch channel is read.
1. Its creator is this bot or a pinned peer's bot user: its newest envelope message whose bot is this bot, or whose app is the pinned app of its `from` machine, must name this machine's current or a former name with this hub's shortname at `from` or `to`.
   That end is the local orch.
   A live orch enters the map with its delivery cursor at the channel's start and its operator cursor and thread floor at its newest message, so messages within retention are delivered once and no operator message already there is handled.
   A retired orch leaves the channel unmapped with one notice in it naming the orch as retired.
2. Its creator's bot belongs to no pin: the newest envelope, whoever posted it, read only to route this notice, must name this hub's shortname at either end.
   Then one notice per channel naming the channel, the creator's app id (`UserAppID` of the channel's creator) and the `from` machine, asking for `approve <machine>` or `approve <machine> <app id>` in that orch channel, is posted in the prime orch's channel.
   When the prime orch has no open channel, it is posted in the open channel of each orch whose button is on; it is also mailed to the prime orch.
   Once the app is approved, the next resync adds the channel.
3. Every other channel is left alone.
Bound: a channel no approved bot made is never delivered from, and a resync reads it only for its notice; a foreign channel's events cost at most one resync per base interval per hub.

`TestBridgeReceive` (tier 0), rows over `fakeSlack`, `fakeMailbox` and `manualClock`:
- a pinned-app message is delivered once from `<machine>::<name>`, and a reread delivers nothing new;
- this bot's own envelope, another human's message, a bot message with no `bot_id`, a bot message whose first line is no envelope, and an envelope whose `to` is another orch are each skipped, delivering nothing and moving the cursor past them;
- a deliver lost on a dropped connection holds the cursor and delivers at the next read;
- a deliver whose reply-to creel refuses is skipped with one notice quoting the reason, and the next message delivers;
- an operator `approve` or `revoke` naming this relay's own machine in a bridge channel, or its former name `Old_Box` typed as `old_box`, pins nothing and answers nothing;
- two hubs of one machine sending id 7 to one orch both deliver;
- another app's message is skipped with one notice, and later pinned messages deliver;
- an unpinned machine holds the cursor with one notice; an operator `approve` after it in the channel is handled and delivers the held messages in order, skipping one past retention;
- under a held message, `approve`, `revoke` and a plain line are each handled once across polls and the `revoke` stands; once both are handled, a reread posts no second notice; a new message held after that `revoke` posts a notice of its new episode;
- an operator `approve` as a thread reply is handled once and answered in its thread;
- a `users.list` refusal on an operator `approve` in a bridge channel holds the operator cursor and is retried at each poll, and after `retryWindow` of clock it is answered in the channel and the operator cursor moves past it;
- a held message stays held past five minutes of clock;
- a message to an orch whose button is off is held until the button is on;
- an envelope naming this machine's former name as `to` is delivered, and as `from` is skipped with one notice;
- a message to a retired orch is skipped with one notice naming it;
- other operator text is answered once by the first-sorting end, and not by the other;
- resync adds a channel made by an approved bot naming this hub's orch, ignoring a human-posted envelope in choosing the local orch, and leaves out one made by another creator or naming another hub; it adds one whose newest envelope names this machine's former name with this hub's shortname;
- the poll round runs a resync at the relay's start and again once the clock passes `bridgePollCeiling`, and not between;
- the poll's waits double while a held message is re-read with nothing new, and return to five seconds after a newly delivered message; a bridge-channel hint and a bridge thread hint each wake the poll at once, but not within `wakeFloor` of the previous round's start;
- resync leaves a retired orch's channel unmapped with one notice, and after a restart posts no second notice;
- an unpinned bot's channel gets one notice in the prime orch's channel naming its app id, and after `approve` the next resync adds it; with no prime channel, the notice goes to each open channel whose button is on and is mailed to the prime orch;
- an adopted or resync-added channel delivers its messages within retention once, and handles neither an old operator `approve` nor an envelope past retention;
- on a machine named `bridge`, a resync reads none of the hub's orch channels.
`TestSocketHolder` (tier 0) gains rows: an event in a bridge-map channel writes that channel's hint into its hub's inbox; repeated events in an unmapped channel within one base interval write one `bridge-resync` hint per live hub under the manual clock, and no `resync` hint (so no orch channel is reread; the read side is pinned by the `bridge` machine row above).
`internal/slackengine/doc.go` gains the receiving half.

**ImpactSummary:** `route`, `takeHints` and `wake` gain bridge branches beside the orch-channel paths, which keep their behaviour; `RunRelay`'s poll round grows the read and resync.

**Commit:** 11: the relay reads bridge channels and delivers to creel
