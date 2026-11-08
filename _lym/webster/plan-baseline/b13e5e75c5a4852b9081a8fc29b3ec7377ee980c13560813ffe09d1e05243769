# Card 12 — outbound-routing

**Create:**
- `internal/slackengine#outboundRoute`
- `internal/slackengine#outbound.channelFor`
- `internal/slackengine#outbound.postTo`
- `internal/slackengine#outbound.postRecovered`
- `internal/slackengine#errNoOpenChannel`
- `internal/slackengine#orchIcon`
- `internal/slackengine#programIcon`

**Delete:**
- `internal/slackengine#outboundText`

**Edit:**
- `internal/slackengine#outbound`
- `internal/slackengine#newOutbound` -> `func newOutbound(geom Geometry, mailbox Mailbox, slack SlackClient, clock Clock, floor *sharedFloor, keeper *channelKeeper, hub Hub) *outbound`
- `internal/slackengine#outbound.read`
- `internal/slackengine#outbound.post` -> `func (out *outbound) post(ctx context.Context, messages []creelengine.Message) error`
- `internal/slackengine#outbound.postMessages` -> `func (out *outbound) postMessages(ctx context.Context, record *outboxRecord, messages []creelengine.Message) error`
- `internal/slackengine#outbound.refused` -> `func (out *outbound) refused(ctx context.Context, channel string, record *outboxRecord, message creelengine.Message, err error) (bool, error)`
- `internal/slackengine#outbound.reportStale` -> `func (out *outbound) reportStale(ctx context.Context, record *outboxRecord) error`
- `internal/slackengine#NewOutbound` -> `func NewOutbound(geom Geometry, mailbox Mailbox, slack SlackClient, clock Clock, keeper *ChannelKeeper, hub Hub) *Outbound`
- `internal/slackengine#RunRelay`
- `internal/slackengine_test#newOutboundHarness`
- `internal/slackengine_test#TestOutboundRead`

**Uses:**
- `internal/slackengine#channelKeeper.ensure`
- `internal/slackengine#channelKeeper.join`
- `internal/slackengine#UpdateChannelMap`
- `internal/slackengine#ReadChannelMap`
- `internal/slackengine#errChannelRefused`
- `internal/slackengine#NewChannelKeeper`
- `internal/slackengine#programSlug`
- `internal/slackengine#orchName`
- `internal/slackengine#slackEscape`
- `internal/slackengine#refusalCap`
- `internal/slackengine_test#newHarness`
- `internal/slackengine_test#fakeSlack`
- `internal/slackengine_test#fakeHub`

**Intent:**
Post each message for `<shortname>:SLACK` to its sender's channel under the sender's name and icon (D8, with D6's routing of a gone beam's mail).

`outboundRoute` replaces `outboundText`: an orch strand routes to its own orch, `agentname.TaskSlug` giving the slug; a program `<shortname>:<slug>:<NAME>` routes to that beam's orch and `<shortname>:<NAME>` to the hub's orch, `programSlug` giving the slug.
It refuses, as today, a non-orch strand, another hub's program and `EXTERNAL`.
The `Sender` carries the creel name as `Username` and `orchIcon` for an orch or `programIcon` for a program; the body is posted escaped with no `[<sender>]` prefix and no answer hint.

`channelFor(orch)` gives the channel a post goes to and the orch whose channel it is:
the hub's orch is ensured through the keeper; a beam's orch reads `Hub.Beams`, and a listed beam's channel is ensured, the first routed mail making it;
a beam the read does not list posts to the prime's channel under the sender's name, without reopening anything;
while the read fails, a beam whose channel the map holds open posts there, and one with none answers `errNoOpenChannel`.
`postMessages` leaves such a message unhandled and goes on with the rest of the batch, recording each one it posts, and returns `errNoOpenChannel` at the batch's end, which releases the read so the waiting mail is delivered again and the outbox record skips what posted.
So a beam with no open channel holds only its own mail, never the prime's or another beam's behind it.
A refused ensure (`errChannelRefused`) is a refusal of the post for the cap; a transient ensure failure releases the read.

The signatures drop `setup`, since no channel comes from it: `post`, `postMessages` and `reportStale` find each channel through `channelFor`, and `refused` is told the channel of the post it counts.
The line about a dropped message posts to that message's channel, or, when its ensure was refused so it has none, to the prime's channel;
with the prime's channel refused too, the relay mails the hub's orch one `[slack relay]` notice carrying the line, which names the message and `lym creel log --to <shortname>:SLACK`.
`outbound.go`'s file header says it posts each message to its sender's channel.
The outbound uses the one keeper `RunRelay` builds (card 11).

`postRecovered` posts with `PostAs` and recovers once from three refusals:
`is_archived` marks the entry archived, ensures a new channel and posts again; `not_in_channel` joins the bot through the keeper and posts again;
`channel_not_found` drops the map entry, ensures the channel anew and posts again.
`postTo` posts a message through `postRecovered`, and each message post Slack refuses with a code counts once toward `refusalCap` through `refused`, and a refused join or ensure after it adds no second count;
a transport error, a server error or a rate limit counts toward no cap, as a post's does today.
The stale line, one count of the hub's stale mail, posts to the prime's channel.
The stale line (`reportStale`) and the dropped-message line (`refused`) post through `postRecovered` under the relay's own name `<shortname>:SLACK` with `programIcon`, so a refusal of either counts toward no message's `refusalCap`.
`read`'s missing-setup error names `lym slack setup --operator <user-id>`.
`RunRelay` builds the outbound with its keeper and hub.

Tests, tier0, rows of `TestOutboundRead` over `newOutboundHarness`, whose keeper is `NewChannelKeeper` over the harness with a `LiveHubs` answering the harness hub alone, and whose map holds `ly:orch` on `C1` and `ly:payments:orch` on `C2`, each entry recording `Operator` `U1` so no post invites anyone:
- the existing posting rows now expect bare escaped bodies posted to `C1` with `username` `ly:orch` and `:robot_face:`, or `ly:BATTEN` and `:gear:`;
- `TextIsEscapedAndBeamMailCarriesTheAnswerHint` becomes: mail from `ly:payments:orch` and `ly:payments:DOBBY` posts to `C2` with no answer hint, the second with `:gear:`;
- mail from a listed beam with no entry creates its channel at that first mail and posts there;
- mail from `ly:gone:orch`, a beam the fake hub does not list, posts to `C1` under `ly:gone:orch`;
- with the beams read failing, a batch of `ly:other:orch`, `ly:orch` and `ly:payments:orch` mail posts the second to `C1` and the third to `C2`, leaves the first unposted and the read unacknowledged, and the redelivered batch posts nothing twice;
- a post refused with `is_archived` creates a new channel and posts there, `not_in_channel` joins `C2` and posts again, and `channel_not_found` drops the entry, creates the channel anew and posts again, each read acknowledging the message;
- a refusal followed by a refused join counts one toward the cap;
- the stale line's post to `C1` refused once with `not_in_channel` joins `C1` and posts the line, clearing the record's stale count;
- a read with `setup.json` removed returns an error naming the setup path and `lym slack setup --operator <user-id>`, and posts nothing;
- a listed beam whose ensure is refused, its name held by a seeded channel of another bot, counts one refusal per read and, at the fifth, drops the message with the line posted to `C1`; with the prime's ensure refused too, the line reaches the hub's orch as a `[slack relay]` notice;
- a listed beam whose ensure fails on a transport error releases the read, counts no refusal and posts at the next read once the ensure stands.
`MessageSlackRefusesFiveTimesIsDroppedWhetherOrNotTheReplacementLinePosts` asserts that the line posts to `C1`, the message's channel, and counts the `chat.postMessage` calls `logged` gives in place of `callCount`;
`MailOlderThanADayIsAcknowledgedUnpostedWithOneCountLine` asserts that the stale line posts to `C1` with stale mail from `ly:payments:orch` among it.

**ImpactSummary:** confined to the outbound read of `internal/slackengine` and its test; the constructors' re-signs reach only `RunRelay` and the test export.
