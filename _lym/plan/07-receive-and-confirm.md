# Card 7 — receive-and-confirm

**Create:**
- `plan:internal/creelengine#VerbDeliver` -> `const VerbDeliver = "deliver"`
- `plan:internal/creelengine#VerbConfirm` -> `const VerbConfirm = "confirm"`
- `plan:internal/creelengine#HubMail` -> `type HubMail struct`
- `plan:internal/creelengine#DeliverAnswer` -> `type DeliverAnswer struct`
- `plan:internal/creelengine#Store.Deliver` -> `func (s *Store) Deliver(mail HubMail) (DeliverAnswer, error)`
- `plan:internal/creelengine#Store.Confirm` -> `func (s *Store) Confirm(mail HubMail) bool`
- `plan:internal/creelengine#Store.heldLocked` -> `func (s *Store) heldLocked(sendingHub string, mail HubMail) (held *Message, clash string)`
- `plan:internal/creelengine#Store.storeInboundLocked` -> `func (s *Store) storeInboundLocked(sendingHub string, mail HubMail, trusted bool) (int64, error)`
- `plan:internal/creelengine#Store.dropLocked` -> `func (s *Store) dropLocked(shortname, sendingHub string, mail HubMail, reason string) error`
- `plan:internal/creelengine#Store.trustedSender` -> `func (s *Store) trustedSender(sender string) bool`
- `plan:internal/creelengine#dropWindow` -> `type dropWindow struct`
- `plan:internal/creelengine#daemon.serveHubCall` -> `func (d *daemon) serveHubCall(conn net.Conn, req Request) error`
- `plan:internal/creelengine_test#hubPair` -> `type hubPair struct`
- `plan:internal/creelengine_test#newHubPair` -> `func newHubPair(t *testing.T) *hubPair`
- `plan:internal/creelengine_test#hubPair.startSending` -> `func (p *hubPair) startSending()`
- `plan:internal/creelengine_test#hubPair.startReceiving` -> `func (p *hubPair) startReceiving()`
- `plan:internal/creelengine_test#hubPair.unpark` -> `func (p *hubPair) unpark()`
- `plan:internal/creelengine_test#hubFake` -> `type hubFake struct`
- `plan:internal/creelengine_test#listenHubFake` -> `func listenHubFake(t *testing.T, r *rig, answer func(req creelengine.Request) (creelengine.Response, string)) *hubFake`
- `plan:internal/creelengine_test#TestDaemon_CrossHubDeliver` -> `func TestDaemon_CrossHubDeliver(t *testing.T)`

**Edit:**
- `internal/creelengine#Request`
- `internal/creelengine#Response`
- `internal/creelengine#daemon.serve`
- `internal/creelengine#Store`
- `internal/creelengine#OpenStore`
- `internal/creelengine/doc.go`

**Uses:**
- `plan:internal/testkit/creelfake#Hubs`
- `internal/creelengine_test#newRig`
- `internal/creelengine_test#rig`
- `internal/creelengine_test#rig.start`
- `internal/creelengine_test#rig.writeJournal`
- `internal/creelengine_test#rig.rawExchange`
- `internal/creelengine_test#newEnv`
- `internal/creelengine#Store.FailJournalWrites`
- `internal/creelengine#Client`
- `internal/creelengine#NewClient`

**Intent:** Serve the `deliver` and `confirm` verbs: the receiving daemon confirms the sender with the sending daemon before anything reaches an agent, journals a drop when the confirm says no, and stamps the trust at arrival (D3, D5).

**What this lets through.** `deliver` and `confirm` skip the sender resolution every other verb passes, since the calling daemon runs in no pane and the confirm alone settles who sent.
So any process of the user may call them, and what bounds that is: a `deliver` stores nothing until the sending daemon's own journal confirms the full tuple, and a `confirm` answers only yes or no and never sends text.
D3's bounds stand as the record states them, and the package doc states them too.

`HubMail` is one request's mail: sending id, sender, receiver, reply-to id, expects-reply mark, post time, body and body hash, the SHA-256 in hex; `confirm` carries no body.
`Deliver` ignores a body hash the request supplies and computes it from `Body` for steps 3, 4 and 6, so a replayed tuple with an altered body gets `no`.
`Request` gains `Mail *HubMail` under `json:"mail,omitempty"`.
`Response` gains `Delivery` (`accepted` or `refused`), `Reason` and `Confirmed` (`yes` or `no`), and `ID` carries the receiver's id.
`daemon.serve` sends the two verbs to `serveHubCall` before `resolve`, which is why a live strand with no known pane process cannot refuse a `deliver`, and these verbs skip `recordHub`.
A request with no `Mail` is refused as `OK: false`, which a hand-written line reaches.
A daemon with a nil seam refuses them as `OK: false` too: a fail-loud wiring guard, since production always passes a seam (card 4).

`Store.Deliver` runs D3's steps in order, and holds the store lock across no call to another daemon:
1. Refused with the reason: a sender that is no strand name, a sender of this hub's shortname, a receiver that is no strand name, a line over the cap by `validateStampedLine`, which also refuses an empty body.
2. `hubs.Lookup` of the sender's shortname, before the lock; an error matching `ErrHubUnresolved` answers `refused` with D1's text, any other error is returned, the transient answer, and no path in the request is read.
3. Under the lock, `heldLocked` finds a held message by that hub path and sending id.
   All of sender, `DeliveredTo` against the receiver, reply-to id, mark, `RemotePosted` to the nanosecond and body hash matching answers `accepted` with the held id and does nothing more, whoever holds the message now.
   Any differing is an id clash, answered `refused` with the reason `creel: id clash: this hub holds ids <lo>-<hi> from that hub; a post succeeds once its ids pass <hi> or those messages pass the seven-day retention`, which fits D2's 200-byte cut whole.
   Only then `checkReceiverLocked` runs over strands read before the lock, and its refusal answers `refused`; a failed strand listing returns an error, which `serveHubCall` writes as `OK: false`, a transient answer to the sender.
4. One `confirm` to the resolved hub, through `hubs.Geometry` and `NewClient` with a nil spawn, so it starts no daemon, under a named 10-second timeout on the store's clock.
5. `Store.Confirm` on the sending side answers yes only for an `outbound` entry in the pending state whose id, sender, receiver, reply-to, mark, post time and body hash all match, and no otherwise.
6. On yes, `trustedSender` reads `hubs.TrustedHubs` before the lock, then under the lock `heldLocked` runs again and an id clash is refused as in step 3; a matching held message answers `accepted` with its id; otherwise `storeInboundLocked` journals `eventInbound` under a new id with the sending hub's path and id, `Trusted`, `RemotePosted` and `DeliveredTo`, delivers it as mail through `deliverMailLocked`, and answers `accepted`.
   On no, `heldLocked` runs under the lock too: a matching held message answers `accepted` with its id, since an overlapping retry stored it and journaled its remote delivery, so a genuine message is never counted as a drop.
   Otherwise `dropLocked` runs and the answer is `refused`.
   A failed or timed-out confirm returns an error, the transient answer, and journals no drop.
   Any confirm answer other than yes or no counts as failed.

`trustedSender` is true when the daemon's platform is `linux`, the sender's shortname is in the list, and `agentname.Parse` gives the role exactly `agentname.RoleOrch`, so `<s>:orch` and `<s>:<slug>:orch` pass and `orch-2` does not.
A `TrustedHubs` error logs one Warn and answers false.

`dropLocked` journals `eventDropped` under a new id with the shortname, the path, the claimed strand as `From`, the receiver as `To` and `Reason`.
A failed journal write returns its error, so the `deliver` gets the transient answer and no notice goes out, and the operator's rule that every drop is journaled holds on the retry.
It then applies the notice window in `Store.dropWindows`, a map keyed by the sending hub path step 2 resolved, so it holds at most one entry per hub of the machine record whatever strands a flood claims.
Each of the first five drops of an hour posts a `CREEL` notice to `<shortname>:orch` of this hub, formed by `agentname.Format`, naming the sending hub's shortname and path, the claimed strand, the receiver, the reason and `lym creel log --from <claimed strand>`.
The fifth also says further drops from that hub this hour are journaled without a notice and names `lym creel log --since 1h`.
The first drop past the hour opens a new window, and its notice also counts the drops journaled since the last notice.
A drop sends the notice, never a timer, and the window is in memory alone.
A notice that fails to store is logged and undoes nothing.

`hubPair` holds two rigs with shortnames `ly` and `lyt`, each with a `creelfake.Hubs` holding both, each test-process sender set up as `newRig` does, and the `running` handle of each daemon it started.
`newHubPair` starts nothing, so a row writes journal lines first, starts a side with `startSending` and `startReceiving`, stops it through its handle, or binds `listenHubFake` on a side's socket instead.
`newHubPair` also sets a geometry error for the receiving hub in the sending rig's `Hubs`, which parks card 8's worker on its retry wait, so a row's hand-sent `deliver` never races it; `unpark` clears it.
`listenHubFake` serves that socket with `answer`, which may block until the row releases it, and counts the requests by verb.
`answer` returns a `Response` and a raw line: a non-empty raw line is written verbatim with a line end in place of the `Response`, so a row can reach the client's JSON decoder, and otherwise the fake writes the `Response` with the `HubPath` the answer set, or the rig's hub path when the answer left it empty.

`TestDaemon_CrossHubDeliver` is tier0 over real sockets, as `TestDaemon_Protocol` is, and each row posts on the sending rig and sends the `deliver` by hand through `rawExchange`.
Rows:
- A hand-written `deliver` naming a sender the sending daemon never sent is refused, reaches no agent, leaves a drop naming hub, claimed strand, receiver and reason, sends one notice to the receiving orch, and `log --from` shows it.
- A `deliver` of a real pending id differing in each of sender, receiver, reply-to, mark and post time is refused as a drop, each answered no by the tuple alone.
- A replayed tuple with an altered body and its true hash supplied is refused and stores nothing.
- A flood from one hub, one claimed strand and many, yields five notices, the fifth naming `lym creel log --since 1h`; the first drop past the hour, by the fake clock, sends one counting the drops since.
- A second `deliver` of a stored message answers the same id and stores nothing, also after the holder forwarded it, and after the receiver's strand left the listing.
- A held hub path and id differing in each of sender, receiver, reply-to, mark, post time and body hash is refused naming the clash and the range, and stores nothing; the post-time row repeats all else, as a re-created hub's message would, and is never answered `accepted` unstored.
- A confirm after the remote delivery record, one after a return record, and one with a differing body hash answer no; those two records are journal lines written before `startSending`.
- The sending daemon down: the `deliver` answers `OK: false`, `Hubs.Spawns` stays 0, and nothing is stored or dropped.
- A confirm the `hubFake` holds past 10 seconds of the fake clock, and one answering neither word, each answer `OK: false` with nothing stored or dropped.
- While the `hubFake` holds a confirm, an `inbox` on the receiving daemon answers, so no store lock is held across the call.
- Two overlapping deliveries of one message, the `hubFake` holding both confirms until both arrive, store it once and answer one id.
- Two overlapping deliveries of one id with differing bodies, held the same way, store the first and refuse the second as a clash at step 6.
- A retry that stored while another attempt's confirm was held, whose held confirm then answers no, answers `accepted` and journals no drop.
- Two hubs delivering to each other at once both complete.
- A `deliver` while a live strand of the receiving hub has `PanePID` 0 is accepted.
- `<s>:orch` and `<s>:<slug>:orch` of a listed hub store trusted; a non-orch strand, an `orch-2` strand and the orch of an unlisted hub store untrusted; a `TrustedHubs` error stores untrusted; `goos` other than `linux` stores untrusted; a list changed by `SetTrusted` between two arrivals applies to the second.
- Refusals at step 1 and 2: a program sender, a sender of the receiver's own shortname, a program receiver, an over-cap line, an empty body, a sending shortname unknown to `Hubs`, one added twice, and an unknown receiver after the duplicate check.
- A `Lookup` error not matching `ErrHubUnresolved`, a failed strand listing and a request with no `Mail` each answer `OK: false` and leave no drop.
- On a `newEnv` store with `SetHubs`, and a `hubFake` answering yes: a `Deliver` whose journal write fails through `FailJournalWrites` returns an error and stores nothing, and the retry stores the message and answers `accepted`; this is D3's receiver that confirmed and did not store.
- On the same store with a `hubFake` answering no, a failing journal write returns an error and sends no notice.

**ImpactSummary:** Two new verbs handled before `resolve`, with every other verb's path untouched; `Request` and `Response` gain optional keys an older client ignores, and an older daemon answers `deliver` with its unknown-verb refusal, which card 8 retries.
