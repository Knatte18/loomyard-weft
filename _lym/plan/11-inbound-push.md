# Card 11 — inbound-push

**Create:**
- `plan:internal/slackengine#inbound.start` -> `func (in *inbound) start()`
- `plan:internal/slackengine#inbound.drain` -> `func (in *inbound) drain(ctx context.Context) (bool, error)`
- `plan:internal/slackengine#inbound.retryOwed` -> `func (in *inbound) retryOwed(ctx context.Context) bool`
- `plan:internal/slackengine#inbound.readChannel` -> `func (in *inbound) readChannel(ctx context.Context, setup Setup, orch string, scan bool) (bool, error)`
- `plan:internal/slackengine#inbound.readThread` -> `func (in *inbound) readThread(ctx context.Context, setup Setup, orch, thread string) (bool, error)`
- `plan:internal/slackengine#owedRead` -> `type owedRead struct`
- `plan:internal/slackengine#parseHint` -> `func parseHint(name string) (channel, thread string, ok bool)`
- `plan:internal/slackengine#fileLines` -> `func fileLines(files []File) string`
- `plan:internal/slackengine#inbound.Start` -> `func (in *inbound) Start()`
- `plan:internal/slackengine#inbound.Drain` -> `func (in *inbound) Drain(ctx context.Context) (bool, error)`
- `plan:internal/slackengine#inbound.RetryOwed` -> `func (in *inbound) RetryOwed(ctx context.Context) bool`
- `plan:internal/slackengine_test#harness.hint` -> `func (h *harness) hint(name string)`
- `plan:internal/slackengine_test#harness.sayIn` -> `func (h *harness) sayIn(channel, ts, text string)`
- `plan:internal/slackengine_test#harness.drain` -> `func (h *harness) drain() (bool, error)`
- `plan:internal/slackengine_test#harness.mustDrain` -> `func (h *harness) mustDrain() bool`
- `plan:internal/slackengine_test#TestInboundRead` -> `func TestInboundRead(t *testing.T)`

**Delete:**
- `internal/slackengine#inbound.poll`
- `internal/slackengine#inbound.Poll`
- `internal/slackengine#inbound.cursorFor`
- `internal/slackengine#readCursor`
- `internal/slackengine#writeCursor`
- `internal/slackengine#cursorRecord`
- `internal/slackengine#route`
- `internal/slackengine#mentionedSlug`
- `internal/slackengine#CursorFileName`
- `internal/slackengine_test#harness.setChannel`
- `internal/slackengine_test#harness.poll`
- `internal/slackengine_test#harness.mustPoll`
- `internal/slackengine_test#TestInboundPoll`

**Edit:**
- `internal/slackengine#inbound`
- `internal/slackengine#inbound.handle` -> `func (in *inbound) handle(ctx context.Context, setup Setup, orch, channel string, message Message) (bool, error)`
- `internal/slackengine#inbound.setPresence` -> `func (in *inbound) setPresence(ctx context.Context, channel string, message Message, presence Presence) (bool, error)`
- `internal/slackengine#inbound.status` -> `func (in *inbound) status(ctx context.Context, channel string, message Message) (bool, error)`
- `internal/slackengine#inbound.forward` -> `func (in *inbound) forward(ctx context.Context, orch, channel string, message Message, text string) (bool, error)`
- `internal/slackengine#inbound.retryOrGiveUp` -> `func (in *inbound) retryOrGiveUp(ctx context.Context, channel string, message Message, reason string, err error) (bool, error)`
- `internal/slackengine#inbound.answer` -> `func (in *inbound) answer(ctx context.Context, channel, text string)`
- `internal/slackengine#inbound.readFailed`
- `internal/slackengine#NewInbound` -> `func NewInbound(geom Geometry, mailbox Mailbox, slack SlackClient, hub Hub, clock Clock, keeper *ChannelKeeper) *Inbound`
- `internal/slackengine#runInbound`
- `internal/slackengine#RunRelay`
- `internal/slackengine#RelayDeps`
- `internal/slackengine#Geometry`
- `internal/hubgeom#SlackGeometry`
- `internal/hubgeom#TestSlackGeometry`
- `internal/slackcli#runRelay`
- `internal/slackcli#newHarness`
- `internal/slackengine_test#newGeometry`
- `internal/slackengine_test#harness`
- `internal/slackengine_test#newHarness`
- `internal/slackengine_test#harness.startAt`
- `internal/slackengine_test#harness.cursor`
- `internal/slackengine_test#harness.wantCursorTS`
- `internal/slackengine_test#newRelayRun`
- `internal/slackengine_test#TestRunRelay`

**Uses:**
- `plan:internal/slackengine#ChannelKeeper`
- `plan:internal/slackengine#channelKeeper.join`
- `plan:internal/slackengine#newChannelKeeper`
- `internal/slackengine#ReadChannelMap`
- `internal/slackengine#UpdateChannelMap`
- `internal/slackengine#ChannelEntry.Advance`
- `internal/slackengine#ChannelEntry.AdvanceThread`
- `internal/slackengine#ChannelEntry.ThreadStart`
- `internal/slackengine#ChannelEntry.PruneThreads`
- `plan:internal/slackengine#resyncHint`
- `internal/slackcli#liveHubGeometries`
- `internal/hubgeom/doc.go`
- `internal/slackengine#inboundText`
- `internal/slackengine#slackTimestamp`
- `internal/slackengine#isRateLimit`
- `internal/slackengine#isAPIError`
- `internal/slackengine_test#fakeSlack`

**Intent:**
Read each channel from Slack after its cursor when a hint, a resync or the start asks, and route each operator message by the channel it was written in (D2, D7).
The single polled channel, its cursor file and the `@<slug>` routing go.

Hints: `drain` lists `InboxDir` once per wake from one goroutine, skipping names that start with a dot.
`resync` owes every open channel a read with its 24-hour thread scan.
`parseHint` reads `<channel>` or `<channel>_<thread ts>`; a hint whose name does not parse, or whose channel the relay's own map does not hold, is removed unread with a Debug line and reads nothing from Slack.
Each hint is removed before the read it triggers, so a hint written during that read is a new file read at the next wake.
`start` owes every open channel a read with its thread scan; card 13 calls it at the relay's start.

Reads: `readChannel` calls `History` after the entry's cursor and handles each message oldest first, advancing the cursor through `UpdateChannelMap` after each handled message.
With `scan` it also reads `History` from 24 hours before now and, for each top-level message whose `LatestReply` is past `ThreadStart`, reads those replies;
`readThread` calls `Replies` after `ThreadStart`, skips the message whose timestamp is the thread's own and every `thread_broadcast` (the top-level read forwards it), and advances the thread cursor past each.
Each read prunes the entry's thread cursors (`PruneThreads`).
A read refused with `not_in_channel` during a start or resync read joins the bot through the keeper's `join` and reads again once.
A read that fails on a transport error, a server error or a rate limit stays owed in the in-memory set of `owedRead` (orch, thread, scan, and the earliest retry time after a rate limit's retry wait);
`retryOwed` makes each owed read whose time has come and reports whether any stood with something handled; card 13 calls it at each wake from the outbound read's wait.
The set dies with the relay, whose next start reads every open channel.
Messages: `handle` forwards a message of no subtype, `file_share` or `thread_broadcast` from the setup's operator, read from `setup.json` at each drain, and skips any other subtype, a bot's message and anyone else's.
The text goes to the orch the map gives for the channel, its `@<slug>` prefix forwarded as plain text, followed by one line `(file not relayed: <name>)` per file (`fileLines`); lym never downloads a file.
`away`, `back` and `status` work in any of the hub's channels, set or read the hub's one presence, and are answered in the channel they were written in, as every answer and the paused line are.
A message that cannot be handled holds its channel's later messages until the retry window ends, as today.
An `answer` Slack refuses with `not_in_channel` joins the bot through the keeper's `join` for the orch the map gives for the channel, and posts once more.
`readFailed`'s notice names `lym slack login` and the manifest in `docs/slack-app.md`, not `lym slack setup`.
Its once-per-episode mark is kept per orch, so one channel's standing refusal mails no second notice when another channel's read stands.
A drain that finds no `setup.json` returns an error naming its path and `lym slack setup --operator <user-id>` from the operator's terminal.
`inbound.go`'s file header says it reads the hub's channels on a hint, a resync or the start.

Wiring: `RelayDeps` gains `LiveHubs`, which `internal/slackcli`'s `runRelay` fills with `liveHubGeometries`;
`newRelayRun` sets it to answer the harness hub, and the inbound `newHarness` builds its keeper with a `LiveHubs` answering the harness hub.
`RunRelay` builds one keeper, which the inbound here, the outbound (card 12) and the wake loop (card 13) share, so its per-orch lock, its bot user and its refused-retire memory hold across all three.
Until card 13, `runInbound` calls `start` once and then `drain` and `retryOwed` after each of its waits.
`Geometry.CursorPath` and `CursorFileName` go with their builders `hubgeom.SlackGeometry`, `TestSlackGeometry`, `newGeometry` and `internal/slackcli`'s `newHarness`; the old `cursor.json` stays on disk unread.

Tests, tier0, `TestInboundRead` replaces `TestInboundPoll`, over `newHarness`, whose map holds `ly:orch` on `C1` and `ly:payments:orch` on `C2`:
- a `C2` hint forwards the operator's message to `ly:payments:orch` as `[slack] …`, and an `@payments x` text in `C1` reaches `ly:orch` unchanged;
- a `C2` hint file holding the text `deploy prod` forwards only the text Slack returns for `C2`, and nothing at all when `C2` holds no new message;
- a `resync` hint reads every open channel after its cursor and, through its 24-hour scan, a reply past a thread cursor; a resync read failing on a server error is made again by `RetryOwed` with its scan; a resync read refused with `not_in_channel` joins and reads again;
- a drain with `setup.json` removed returns an error naming `lym slack setup --operator <user-id>`;
- `C1` refused with `invalid_auth` across two drains while `C2` reads mails one notice;
- a hint naming `C9`, one named `junk_` and one named `.tmp` read nothing from Slack, and the first two are removed;
- a hint written by `fakeSlack`'s `onCall` during the read stays in the inbox and is read at the next drain;
- a hinted read failing with a transport error, an HTTP 500 and a `*RateLimitError` forwards nothing, and `RetryOwed` with no new hint forwards the message once it stands; the rate-limited one waits its retry wait;
- a failed `Start` read is made again by `RetryOwed` with its thread scan, forwarding a reply;
- a message hinted after an earlier hint was lost forwards both; a message both hinted and read by `Start` forwards once;
- `Start` forwards the top-level messages after each cursor and the replies past each thread cursor of the last 24 hours' messages;
- a thread cursor 25 hours old is pruned with the floor raised, a later reply in it forwards and an earlier one does not;
- an entry whose floor sits at its newest message, as an adoption leaves it, forwards no earlier reply at `Start`;
- `away`, `back` and `status` written in `C2` set and read the hub's presence and are answered in `C2`;
- a `status` answer in `C2` refused once with `not_in_channel` joins `C2` through the keeper and posts the answer;
- an operator thread reply forwards to the channel's orch; a `thread_broadcast` forwards once whether its channel hint or a `Start` that reads both its channel and its thread finds it;
- a `file_share` forwards its text and one `(file not relayed: <name>)` line per file; a `channel_join`, another user's and a bot's message are skipped;
- a `Start` read refused with `not_in_channel` joins and reads again;
- a changed operator in `setup.json` is the one forwarded at the next drain;
- the kept rows of `TestInboundPoll` move over as rows of the hinted read: the markup, the body cap, the paused answer, the presence retry and give-up rows, the escaped answer, the refused answer, and one refusal notice per episode, now naming `docs/slack-app.md`.
The harness's `startAt` writes the prime entry's cursor, `cursor` and `wantCursorTS` read the map, `hint` writes an empty hint into the inbox, and `sayIn` adds an operator message to a channel.
`TestRunRelay` drops the row `InboundWaitDoublesToTheCeilingResetsOnActivityAndARateLimitRaisesTheFloor`, whose poll is gone; its other rows run over the start's reads.

**ImpactSummary:** rewrites the inbound side of `internal/slackengine` and its tests; outside it, `Geometry.CursorPath`'s removal reaches `hubgeom.SlackGeometry` and its test and `internal/slackcli`'s test harness, and `RelayDeps.LiveHubs` reaches `runRelay`.
