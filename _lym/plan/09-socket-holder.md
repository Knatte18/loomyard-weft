# Card 9 — socket-holder

**Create:**
- `internal/slackengine#socketHolder`
- `internal/slackengine#newSocketHolder`
- `internal/slackengine#socketHolder.run`
- `internal/slackengine#socketHolder.serve`
- `internal/slackengine#socketHolder.route`
- `internal/slackengine#socketHolder.readLive`
- `internal/slackengine#socketHolder.payOwed`
- `internal/slackengine#socketHolder.notice`
- `internal/slackengine#writeHint`
- `internal/slackengine#hintName`
- `internal/slackengine#resyncHint`
- `internal/slackengine#pingInterval`
- `internal/slackengine#pongTimeout`
- `internal/slackengine#envelopeLimit`
- `internal/slackengine#reconnectFloor`
- `internal/slackengine#stableConnection`
- `internal/slackengine#SocketHolder`
- `internal/slackengine#NewSocketHolder`
- `internal/slackengine#socketHolder.Run`
- `internal/slackengine_test#fakeSocket`
- `internal/slackengine_test#fakeSocket.Read`
- `internal/slackengine_test#fakeSocket.Ack`
- `internal/slackengine_test#fakeSocket.Ping`
- `internal/slackengine_test#fakeSocket.Close`
- `internal/slackengine_test#fakeDialer`
- `internal/slackengine_test#fakeDialer.dial`
- `internal/slackengine_test#manualClock`
- `internal/slackengine_test#newManualClock`
- `internal/slackengine_test#manualClock.Now`
- `internal/slackengine_test#manualClock.After`
- `internal/slackengine_test#manualClock.Advance`
- `internal/slackengine_test#manualClock.pending`
- `internal/slackengine_test#TestSocketHolder`

**Extends:**
- `cmd/lym#unboundedWaitFiles`

**Uses:**
- `internal/slackengine#Socket`
- `internal/slackengine#Envelope`
- `internal/slackengine#Dialer`
- `internal/slackengine#LiveHubs`
- `internal/slackengine#ReadChannelMap`
- `internal/slackengine#ChannelMap.OrchOf`
- `internal/slackengine#updateRelayState`
- `internal/slackengine#relayNoticePrefix`
- `internal/slackengine#orchName`
- `internal/slackengine#waitCeiling`
- `internal/slackengine_test#fakeMailbox`
- `internal/slackengine_test#fakeSlack`

**Intent:**
Add the machine's one Socket Mode connection (D1) in a new file `internal/slackengine/holder.go`; card 13 starts it from `RunRelay`.

Lock: `run` takes the machine socket lock at `geom.SocketLockPath` with `lock.AcquireWriteLock`, which blocks in its goroutine without waking until the holder's relay ends; it then holds the lock until ctx ends and releases it.
A ctx that ends during the block leaves the goroutine waiting, and it releases the lock at once when it gets it.

Connections: with the lock held, `run` dials, `serve`s the socket until it fails, closes it, waits on the clock and dials again.
The wait starts at one second, doubles to `waitCeiling` (sixty seconds), and returns to one second only after a connection that served at least `stableConnection`.
A dial that fails with an `*APIError`, a refused or disabled app, or a `disconnect` whose reason is `link_disabled`, mails the holder hub's orch one `[slack relay]` notice per episode naming `lym slack login`; the next connection that serves a minute ends the episode.

Serving: after each connection opens, the holder writes a `resync` hint into every live hub's inbox, overwriting any there.
One goroutine reads envelopes; the serving loop waits on them, on the clock's `pingInterval` and on ctx.
At each ping it races `Ping` against the clock's `After(pongTimeout)`, so the ten-second bound runs on the injected clock; a failed ping, or no pong before that wait ends, ends the connection.
Each pong stamps `LastSlackOK` in the holder hub's relay state.
A `disconnect` envelope or a read error ends the connection.
Each ping also reads the live-hub list again and pays every owed resync hint (`payOwed`).

Routing (`route`) of an `events_api` envelope runs in a goroutine of its own, off the serving loop, which waits for its answer or the clock's `After(envelopeLimit)`:
a route that has not answered when that wait ends is passed the limit, so a map lock or a hint write that never returns stalls neither the pings nor the other hubs' events.
The envelope is then acknowledged and a resync hint owed to the hub the route found, or to every hub of the last live-hub read that stood when it found none yet; the stalled goroutine's later answer is dropped.
It reads `LiveHubs` and every live hub's map at `ChannelMapPath` and finds the hub whose map holds the event's channel.
It writes the hint `hintName` gives, `<channel>` or, for a thread reply that is no `thread_broadcast`, `<channel>_<thread ts>`, into that hub's `InboxDir`, then acknowledges.
`writeHint` creates an empty temp file in the hub's Slack directory, outside the inbox, then renames it into the inbox, creating the inbox when missing; a hint carries no text.
That temp file departs from `PATTERN-lym-files`' "temp under `lymtemp`", as the overview's Shared Decisions record: D1 puts it in the hub's `.lym/slack/` so the rename stays on the inbox's filesystem.
A failed hint write, or routing that passed `envelopeLimit`, acknowledges and owes that hub a resync hint.
An event no map holds is acknowledged and dropped with a Debug line while every map reads; while any live map cannot be read or decoded, an event no readable map holds is left unacknowledged for Slack's retry.
While the `LiveHubs` read fails every event is left unacknowledged, and the holder hub's orch gets one notice per episode carrying the seam's error as it is, which names the path, and the way forward when the error carries one.
That notice is first posted to the holder hub's prime channel when its own map reads and holds it.
A read that stands ends the episode and owes every hub it names a resync hint; so does each hub a read names that the previous one did not.
A hint write failure mails the holder hub's orch one notice per episode naming the inbox directory and the write's error, and the other hubs' events go on.
Hints and owed resyncs are paid at each ping until the write stands.

`NewSocketHolder` in `export_test.go` builds a holder whose hint writer is the given function, nil meaning `writeHint`, and `Run` runs it.
`cmd/lym/daemonwakeups_test.go`'s `unboundedWaitFiles` gains `internal/slackengine/holder.go`, the ping and reconnect loop.

`manualClock` is a `Clock` whose `After` records a pending wait and returns a channel that delivers only once `Advance` moves the time past its deadline; `pending` lists the waits not yet delivered.
Loops under it block until the row advances the time, so a row reads one loop's waits at a quiescent point and no limit drifts while another loop spins, as `fakeClock`'s at-once waits would.
`fakeSocket` delivers each scripted envelope when a row sends it, closes a per-envelope channel at its `Ack`, and its `Ping` answers a scripted error or blocks until its ctx ends; `fakeDialer` closes a channel at each dial.
A row waits on those channels before it advances the clock, so an ack is seen before the next ping.

Tests, tier0, `TestSocketHolder`, over two hubs from `newGeometry` sharing one `SocketLockPath`, `manualClock`, `fakeMailbox`, `fakeSlack` and a scripted `fakeDialer` handing out `fakeSocket`s whose envelopes, read errors, ping errors and acks a row scripts:
- only the holder that took the lock dials; once its ctx ends the second holder dials;
- a holder whose ctx ends while it blocks on the lock another holder keeps takes the lock once it is freed and releases it at once, so a third holder then dials;
- an event for hub B's channel read by hub A's holder writes an empty `<channel>` hint into B's inbox before its ack, a thread reply writes `<channel>_<ts>`, and a `thread_broadcast` writes the channel hint;
- a failing hint writer acks the envelope, owes B a resync hint written at the next ping, and mails hub A's orch one notice naming B's inbox directory and the write's error across two failures, while an event for hub A writes its hint;
  a hint write that then stands, followed by another failure, mails a second notice;
- a hint writer that blocks until the row releases it: the row advances the clock ten seconds, the envelope is acked, B is owed a resync hint written at the next ping, and that ping still runs while the writer stays blocked;
- a route whose `LiveHubs` seam blocks until the row releases it: the row advances the clock ten seconds, the envelope is acked, the row releases the seam, and every hub of the last live-hub read that stood is owed a resync hint written at the next ping;
- the resync hint written after a connection opens fails for B and is written at the next ping;
- an unmapped event is acked while both maps read, and left unacked while B's map holds `{`; every event is left unacked while `LiveHubs` fails with a scripted error carrying the hubs record's path, one notice carries that path, posted first to A's prime channel when A's map holds it, and the read standing at the next ping writes a resync hint into both inboxes and a later hub newly listed gets one;
- each new connection writes a resync hint into every live inbox;
- a `disconnect`, a read error, a failed ping and a `Ping` that blocks past the ten-second `pongTimeout` wait each lead to a new dial;
- with every dial failing, the pending reconnect waits run 1, 2, 4 … 60 seconds, and back to 1 after a connection that served a minute;
- a dial answering an `*APIError` mails one notice naming `lym slack login` across repeated failures, and so does a `disconnect` with reason `link_disabled`;
  after that notice, a connection that serves a minute and then another refused dial mail a second notice;
- a pong stamps `LastSlackOK`.

**ImpactSummary:** a new file and its test inside `internal/slackengine`; the one existing file it extends is the wake-scan list in `cmd/lym`, which gains an entry.
