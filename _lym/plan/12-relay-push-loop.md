# Card 12 — relay-push-loop

**Create:**
- `plan:internal/slackengine#wakeLoop` -> `type wakeLoop struct`
- `plan:internal/slackengine#wakeLoop.run` -> `func (w *wakeLoop) run(ctx context.Context) error`
- `plan:internal/slackengine#wakeLoop.wake` -> `func (w *wakeLoop) wake(ctx context.Context, fromOutbound bool)`
- `plan:internal/slackengine#wakeLoop.watch` -> `func (w *wakeLoop) watch() (<-chan struct{}, func(), error)`
- `plan:internal/slackengine#wakeLoop.diskCheck` -> `func (w *wakeLoop) diskCheck() bool`
- `plan:internal/slackengine#relayStart` -> `func relayStart(ctx context.Context, geom Geometry, keeper *channelKeeper, in *inbound, hub Hub) bool`
- `plan:internal/slackengine#missingTokenExit` -> `func missingTokenExit(ctx context.Context, geom Geometry, deps RelayDeps, err error) error`
- `plan:internal/slackengine#holderRun` -> `type holderRun struct`
- `plan:internal/slackengine#holderRun.start` -> `func (r *holderRun) start(ctx context.Context)`
- `plan:internal/slackengine#holderRun.stop` -> `func (r *holderRun) stop()`
- `plan:internal/slackengine#wakeFloor` -> `const wakeFloor = time.Second`

**Delete:**
- `internal/slackengine#runInbound`
- `internal/slackengine_test#relayRun.awaitOutboundParked`

**Edit:**
- `internal/slackengine#RelayDeps`
- `internal/slackengine#RunRelay`
- `internal/slackengine#sharedFloor`
- `internal/slackengine/doc.go`
- `internal/slackcli#runRelay`
- `internal/slackcli/doc.go`
- `internal/slackcli#TestSlackCLI`
- `internal/slackengine_test#relayRun`
- `internal/slackengine_test#newRelayRun`
- `internal/slackengine_test#relayRun.stop`
- `internal/slackengine_test#TestRunRelay`
- `docs/overview.md`
- `contracts/orchrole/role-slack.md`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `plan:internal/slackengine#newSocketHolder`
- `plan:internal/slackengine#socketHolder.run`
- `plan:internal/slackengine#newChannelKeeper`
- `plan:internal/slackengine#channelKeeper.ensure`
- `plan:internal/slackengine#channelKeeper.inviteOperators`
- `plan:internal/slackengine#channelKeeper.retire`
- `plan:internal/slackengine#inbound.start`
- `plan:internal/slackengine#inbound.drain`
- `plan:internal/slackengine#inbound.retryOwed`
- `plan:internal/slackengine#NewDialer`
- `plan:internal/slackengine#ReadCredentials`
- `plan:internal/slackengine_test#fakeDialer`
- `plan:internal/slackengine_test#fakeSocket`
- `plan:internal/slackengine_test#manualClock`
- `plan:internal/slackengine#HelloTimeout`
- `internal/slackengine#firstRead`
- `internal/slackengine#takeRelayLock`
- `internal/slackengine#inbound.presenceChanged`
- `internal/fswatch#Watch`
- `internal/slackengine_test#holdRelayLock`

**Intent:**
Run the relay on push: it starts the socket holder, wakes on its inbox and presence file, and does the retried work at each wake from the outbound read's wait (D1, D2, D3, D5, D6; `PATTERN-daemon-wakeups` as the record's Constraints word it).

`RelayDeps` gains `Dial`, the Socket Mode seam, and `Watch func(dir string, names ...string) (*fswatch.Watcher, error)`, nil meaning `fswatch.Watch`.
`internal/slackcli`'s `runRelay` reads the credentials with `ReadCredentials` and no refusal, builds the API with the bot token and fills `Dial` with `slackengine.NewDialer(slackengine.DefaultBaseURL, creds.AppToken, client, slackengine.HelloTimeout)`.
`wakeLoop`, `relayStart`, `missingTokenExit` and `holderRun` live in `relay.go`, which stays on `unboundedWaitFiles`, so the wake loop and its on-disk fallback stay under the scan.

`RunRelay` order: reap, take the relay lock, register, record the pid, make the first creel read, all as today; only then `RequireConfigured`.
A refusal there goes to `missingTokenExit`: it mails the hub's orch one `[slack relay]` notice carrying the refusal, which names the missing kind and `lym slack login`, or `lym slack setup --operator <user-id>`;
with a bot token and a map holding the prime's channel it first posts the same line there; with no prime entry and a `setup.json` still carrying an older binary's `channel_id`, it posts it to that channel with the bot token, the only read of that key; then the relay returns the refusal and exits.
The line adds `then lym slack start` to its way forward, since the relay has exited and neither the login nor a dobby tick while presence is `terminal` starts it again.
Then `relayStart` ensures the prime's channel, and every listed beam's when presence is `slack`, and calls `inbound.start` and `retryOwed`; a prime ensure that was refused or failed on a transient error is remembered for the outbound wakes.
`holderRun` starts the socket holder in its own goroutine under its own context; the outbound loop and `wakeLoop.run` follow.

`wakeLoop` creates the inbox directory, then watches it whole and the Slack directory for the presence file alone, so the relay's records, outbox, map, log and lock wake nothing.
A file event wakes it, but never within `wakeFloor` of the previous wake's start: an event inside that second waits the rest of it on the clock.
`wake` reads `setup.json`, drains the inbox, reads presence and, on `slack` after `terminal`, ensures every listed beam's channel, and retires the channels of beams a `Hub.Beams` read that stood no longer lists.
A failed beams read retires nothing, and a retire Slack refused is retried only from an outbound wake.
The outbound loop's `afterRead`, after its last-healthy stamp, only signals `wakeLoop` and waits for its answer; `wakeLoop`'s own goroutine does the outbound wake, so one goroutine reads the channels and holds the inbound's owed set and episodes, as D2 asks.
The outbound wake makes the owed reads again (`retryOwed`), invites the setup's operator where an entry records another or none (`inviteOperators`), retries a refused retire and a remembered prime ensure, then runs the binary check.
None of that work shortens the outbound read's wait, so with no outbound mail the retries back off to sixty seconds with it.
When `Watch` fails at open, `wakeLoop` checks the inbox and the presence file on disk instead (`diskCheck`), waiting from one second and doubling to sixty while it finds nothing, and back to one second after it finds a hint.

Re-exec: the binary check's `beforeExec` stops the holder, which closes its connection and releases the machine socket lock, and releases the relay lock; the exec's image waits its turn for both again.
A failed exec retakes the relay lock as today and starts the holder again, which rejoins the socket lock wait; a relay lock kept by another relay ends this one.
`runInbound` goes.
`relay.go`'s file header, `backoff.go`'s and `sharedFloor`'s comment describe the outbound loop and the wake loop in place of the two polling loops.

Docs: `internal/slackengine`'s and `internal/slackcli`'s package documentation and `docs/overview.md`'s slack bullet and package lines describe the push route, one channel per orch named by hub, machine and orch, the two tokens and the app manifest in `docs/slack-app.md`, and that `stop` archives nothing.
`contracts/orchrole/role-slack.md` says each orch's posts reach its own channel under its own name and the operator answers in that channel or a thread of it, in place of the `@<slug>` answer hint.
`pattern/PATTERN-daemon-wakeups.md`'s Slack bullet lists the loops this card leaves:
the outbound read with the binary check and the retried work after it, from five seconds doubling to sixty;
the holder in `internal/slackengine/holder.go`, its lock wait blocking without waking, its 30-second ping spawning nothing and its reconnect from one second doubling to sixty;
and the inbox and presence watch, never within a second of the previous wake's start, with its on-disk fallback from one second to sixty.
`relay.go` stays on `unboundedWaitFiles`.

Tests, tier0. `newRelayRun`'s mailbox answers each outbound read at once with no mail, in place of blocking, since the binary check follows the read; `relayRun` gains a `fakeDialer`, a real `Watch` and card 8's `manualClock` in place of `fakeClock`, and `stop` ends the run from a wait.
Under `manualClock` each loop blocks at its wait until the row advances the time, so a row reads the pending waits of the loop it asserts at a quiescent point, and no retry window or prune drifts while another loop spins.
`TestRunRelay` rows:
- `HubWithNoTokenOrNoSetupIsRefusedAndTakesNoLock` becomes: a relay with a bot token alone takes the lock, makes its first creel read, mails one notice naming the app-level token, `lym slack login` and `lym slack start`, posts the same line to the prime's channel, and exits; with no map and a `setup.json` holding `channel_id` `C7` it posts the line to `C7`;
  a relay with both tokens and no `setup.json` mails one notice naming `lym slack setup --operator <user-id>` and `lym slack start`, and exits;
- with an owed read failing on a server error at every outbound wake and no mail, the outbound loop's pending waits run 10, 20, 40, 60 and 60 seconds; a refused operator invite at every outbound wake leaves the same sequence;
- a prime ensure failing on a transport error at start is made at the next outbound wake;
- a file event arriving during an outbound wake is drained after that wake ends, never beside it;
- a hint written into the inbox wakes the relay through its file event and forwards the operator's message with no clock wait between;
- writes to the relay state, the outbox, the channel map and the relay log wake nothing, and an event inside a second of a wake's start waits the rest of that second;
- two hubs' relays sharing one socket lock: only one dials, and an event for hub B's channel read by hub A's holder is forwarded by B; once A's relay ends, B dials;
- the prime's channel is ensured at start; with presence `slack` at start every listed beam's is too; a presence write from `terminal` to `slack` ensures them at the next wake; a refused prime ensure is retried at the next outbound wake;
- a refused operator invite, and a `setup.json` naming a new operator, are invited at the next outbound wake with the relay still running;
- an unlisted beam's channel is renamed and archived at a wake; a failing beams read retires nothing; a refused archive is not retried at a file-event wake and is at the next outbound wake;
- a hinted read failing on a server error is forwarded after the next outbound wake with no new hint;
- the binary check's `beforeExec` leaves the socket lock free, and after a failed exec the row waits on `fakeDialer`'s next dial, which comes only once the holder holds the socket lock again;
- with `Watch` failing, a hint on disk is found by the on-disk check, whose waits run 1, 2, 4 seconds and back to 1 after the hint.
The lock and first-read rows stay; `LockKeptByAnotherRelayEndsThisOne` ends the relay from its replaced-binary hook with no outbound park.
`TestSlackCLI`'s `Stop` row asserts the fake Slack saw no request, so `lym slack stop` archives nothing.

**ImpactSummary:** rewrites the relay's run loop in `internal/slackengine` and its relay test; outside it, `internal/slackcli`'s `runRelay` fills the new seams, and five doc files and one stop assertion follow.
