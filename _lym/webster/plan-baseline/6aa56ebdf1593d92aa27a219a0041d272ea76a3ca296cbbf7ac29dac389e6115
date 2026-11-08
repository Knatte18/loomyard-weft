# Card 8 — slack-client-surface

**Create:**
- `plan:internal/slackengine_test#slackCall` -> `type slackCall struct`
- `plan:internal/slackengine_test#fakeSlack.PostAs` -> `func (s *fakeSlack) PostAs(_ context.Context, channel, text string, sender slackengine.Sender) error`
- `plan:internal/slackengine_test#fakeSlack.Replies` -> `func (s *fakeSlack) Replies(_ context.Context, channel, thread, oldest string) ([]slackengine.Message, error)`
- `plan:internal/slackengine_test#fakeSlack.CreateChannel` -> `func (s *fakeSlack) CreateChannel(_ context.Context, name string) (slackengine.ChannelInfo, error)`
- `plan:internal/slackengine_test#fakeSlack.FindChannel` -> `func (s *fakeSlack) FindChannel(_ context.Context, name string) (slackengine.ChannelInfo, bool, error)`
- `plan:internal/slackengine_test#fakeSlack.Join` -> `func (s *fakeSlack) Join(_ context.Context, channel string) error`
- `plan:internal/slackengine_test#fakeSlack.Invite` -> `func (s *fakeSlack) Invite(_ context.Context, channel, user string) error`
- `plan:internal/slackengine_test#fakeSlack.Rename` -> `func (s *fakeSlack) Rename(_ context.Context, channel, name string) error`
- `plan:internal/slackengine_test#fakeSlack.Archive` -> `func (s *fakeSlack) Archive(_ context.Context, channel string) error`
- `plan:internal/slackengine_test#fakeSlack.AuthTest` -> `func (s *fakeSlack) AuthTest(context.Context) (slackengine.Identity, error)`
- `plan:internal/slackengine_test#fakeSlack.failNext` -> `func (s *fakeSlack) failNext(method string, errs ...error)`
- `plan:internal/slackengine_test#fakeSlack.logged` -> `func (s *fakeSlack) logged(method string) []slackCall`
- `plan:internal/slackengine_test#fakeSlack.seedChannel` -> `func (s *fakeSlack) seedChannel(info slackengine.ChannelInfo)`

**Delete:**
- `internal/slackengine#API.PostMessage`
- `internal/slackengine#recordingSlack`
- `internal/slackengine#recordingSlack.stamp`
- `internal/slackengine#recordingSlack.History`
- `internal/slackengine#recordingSlack.Latest`
- `internal/slackengine#recordingSlack.PostMessage`
- `internal/slackengine_test#fakeSlack.PostMessage`

**Edit:**
- `internal/slackengine#SlackClient`
- `internal/slackengine#inbound.answer`
- `internal/slackengine#outbound.reportStale`
- `internal/slackengine#outbound.postMessages`
- `internal/slackengine#outbound.refused`
- `internal/slackengine#runOutbound` -> `func runOutbound(ctx context.Context, out *outbound, clock Clock, log *relayLog, afterRead func())`
- `internal/slackengine#RunRelay`
- `internal/slackengine#RelayDeps`
- `internal/slackengine#RelayState`
- `internal/slackengine#lifetime`
- `internal/slackengine#starter.ensure`
- `internal/slackengine_test#fakeSlack`
- `internal/slackengine_test#postedText`
- `internal/slackengine_test#TestAPI`
- `internal/slackengine_test#TestRunRelay`

**Uses:**
- `internal/slackengine#updateRelayState`
- `internal/slackengine#NewStarter`
- `internal/slackengine_test#fakeSpawn`

**Intent:**
Widen the relay's `SlackClient` seam to every call the push route makes, and stamp the relay's last-healthy time where D1 puts it instead of after every Slack call.

`SlackClient` becomes `History`, `Replies`, `Latest`, `PostAs`, `CreateChannel`, `FindChannel`, `Join`, `Invite`, `Rename`, `Archive` and `AuthTest`, with the signatures card 1 gave `API`; `PostMessage` leaves it.
`API.PostMessage` goes and its callers move to `PostAs` with an empty `Sender`, which sends the same form: `inbound.answer`, `outbound.reportStale`, `outbound.postMessages` and `outbound.refused`, and the `TestAPI` rows that posted.

`recordingSlack` goes: `RunRelay` hands `deps.Slack` to the loops directly.
`runOutbound` calls `afterRead` after each read, before its wait; `RunRelay` passes a closure that stamps `RelayState.LastSlackOK` with the clock's now and logs a failed stamp under `relay-state`.
So every relay stamps at the end of each outbound read, whatever the read found, and card 9's holder also stamps at each pong (D1).
`RelayState.LastSlackOK`'s, `lifetime`'s and `starter.ensure`'s comments name the last-healthy time; the restart cap's logic stays, as the record's notes ask.
`RelayDeps.Slack`'s comment drops the recording.

`fakeSlack` implements the whole seam.
Every call appends a `slackCall` (`Method`, `Channel`, `Arg`, `Sender`) to its log, read with `logged(method)`, and still counts toward `callCount` and runs `onCall`;
a new field `onMethod func(method string)` runs at the start of each call with its method name, outside the lock, so a row can act at one method alone.
`seedChannel` adds a channel a row names with its id, name, `Creator`, `IsArchived` and `IsMember`, as one another user, another bot or an archive left.
`failNext(method, errs...)` queues errors one per call of that method, a nil entry succeeding; `failPosts` stays, now feeding `PostAs`.
`History` returns only a channel's top-level messages: those with no `ThreadTS`, those whose `ThreadTS` equals their `TS`, and `thread_broadcast` messages;
`Replies` returns the messages of one thread after oldest, the parent included, as Slack does.
`CreateChannel` adds a channel with a fresh id `C<n>` under the name, created by `UBOT` and with the bot a member, answering `name_taken` for a name it holds;
`FindChannel` finds by name among created and seeded channels, archived included, with their `Creator`, `IsArchived` and `IsMember`.
`Join` and `Invite` mark membership; `Rename` changes a channel's name, answering `name_taken` for a held name; `Archive` marks it archived; `AuthTest` answers `Identity{UserID: "UBOT", BotID: "B1"}`.
`postedText` gains `Sender`.

Tests, tier0: `TestInboundPoll` and `TestOutboundRead` run unchanged as the refactor's coverage; `TestAPI`'s posting rows assert the same requests through `PostAs`.
Both rows below clear `newRelayRun`'s `blockReadWait` before the run, so each outbound read answers at once with no mail and `afterRead` runs;
each stops the run from an `onWait` hook once the condition it waits for holds, not at the first wait, since both loops take waits on the one clock.
`TestRunRelay`'s `SuccessfulSlackCallStampsLastSlackOK` becomes `OutboundReadStampsLastHealthyWithNoSlackCall`: a relay whose Slack fails every call still stamps `LastSlackOK` after its first outbound read, the hook stopping it once the stamp is set.
A new row seeds presence `slack` and a relay state with `LastSpawnAt` at the clock's now and `NextStartAt` a minute on, as `RelayLifetimeAtOrOverTheHealthyBoundResetsTheCount` seeds its state.
It runs a relay whose outbound reads find no mail until the clock passes eleven minutes after `LastSpawnAt`, ends it,
and asserts that a `NewStarter` over a `fakeSpawn` and the run's clock, called with whileAway, answers `OutcomeStarted` with `StartFailures` 0 (D1's idle-relay criterion); with the stamp moved back to each Slack call it would count one.

**ImpactSummary:** an interface widening inside `internal/slackengine` with one production implementation, `API`, and one fake; four call sites move to `PostAs` with an identical request, and the last-healthy stamp moves from each Slack call to each outbound read.
