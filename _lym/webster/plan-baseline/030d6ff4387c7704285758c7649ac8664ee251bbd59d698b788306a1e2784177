# Card 8 — outbound-worker

**Create:**
- `plan:internal/creelengine#outboundWorker` -> `type outboundWorker struct`
- `plan:internal/creelengine#outboundWorker.run` -> `func (w *outboundWorker) run()`
- `plan:internal/creelengine#outboundWorker.attempt` -> `func (w *outboundWorker) attempt(entry outboundEntry) (answer deliverOutcome)`
- `plan:internal/creelengine#deliverOutcome` -> `type deliverOutcome struct`
- `plan:internal/creelengine#Scheduler.wakeOutbound` -> `func (s *Scheduler) wakeOutbound(hubPath string)`
- `plan:internal/creelengine#Store.SetOutboundHook` -> `func (s *Store) SetOutboundHook(hook func(hubPath string))`
- `plan:internal/creelengine#Store.pendingOutbound` -> `func (s *Store) pendingOutbound(hubPath string) []outboundEntry`
- `plan:internal/creelengine#Store.outboundHubs` -> `func (s *Store) outboundHubs() []string`
- `plan:internal/creelengine#Store.settleDelivered` -> `func (s *Store) settleDelivered(id, remoteID int64) error`
- `plan:internal/creelengine#Store.settleReturned` -> `func (s *Store) settleReturned(id int64, reason string) error`
- `plan:internal/creelengine#quoteAnswer` -> `func quoteAnswer(text string) string`
- `plan:internal/creelengine#daemonRefusal` -> `type daemonRefusal struct`
- `plan:internal/creelengine_test#TestScheduler_Outbound` -> `func TestScheduler_Outbound(t *testing.T)`

**Edit:**
- `internal/creelengine#Scheduler`
- `internal/creelengine#NewScheduler`
- `internal/creelengine#Scheduler.Start`
- `internal/creelengine#Store`
- `internal/creelengine#Store.do`
- `plan:internal/creelengine#Store.postOtherHub`
- `internal/creelengine#Client.exchange`
- `cmd/lym#unboundedWaitFiles`

**Extends:**
- `internal/creelengine/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `plan:internal/creelengine#HubDirectory`
- `internal/creelengine#NewClient`
- `plan:internal/creelengine_test#hubPair`
- `plan:internal/creelengine_test#newHubPair`
- `plan:internal/creelengine_test#hubPair.unpark`
- `plan:internal/creelengine_test#listenHubFake`
- `internal/creelengine_test#newEnv`
- `plan:internal/testkit/creelfake#Hubs`

**Intent:** Deliver what card 6 queues: the sending daemon's outbound worker, with its retries, its start cap, the hour's return and the `CREEL` notice (D2's delivery half).

The worker is in the new file `internal/creelengine/outbound.go`, one per receiving hub path, started by `wakeOutbound` and by `Scheduler.Start` for each of `outboundHubs`, so a pending message survives a sending daemon restart and is delivered after it.
A nil seam starts none.
`postOtherHub` signals the hub path through a list `Store.do` drains to `SetOutboundHook`'s hook after the mutex is free, as it does for receivers.

`run` takes the oldest pending entry of its hub, by id, makes one attempt with no store lock held, and acts on the answer; it retires as `worker.retire` does when none is pending.
Because one worker makes every call to its hub, one at a time, a return can only follow the end of the call in flight, so a return and an `accepted` never cross.
- `accepted`: `settleDelivered` journals `eventRemoteDelivered` with the remote id, and the worker goes straight to the next entry.
- `refused`: `settleReturned` at once, with the reason.
- Anything else is a transient answer: a failed connection, a timeout, `OK: false`, an unknown verb's refusal, an answer with neither word, an answer `Client.exchange` cannot accept, a hub whose geometry cannot be built.
  The worker waits from `outboundRetryBase`, one second, doubling to `outboundRetryCeiling`, a minute, on the Scheduler's clock.
  The wait is capped at what is left of the entry's hour, never below the base, and a wake during it changes nothing, so a new post neither hastens a retry nor passes the start cap.
- A `settleDelivered` or `settleReturned` whose journal write fails takes that same wait and logs one Warn per episode, so the worker never loops below the one-second floor; the next attempt is answered with the held id by D3 step 3, or the same refusal.

A named `outboundDeadline` of one hour runs from the entry's journaled post time, so it counts across a restart and on the injected clock.
At or past it the worker makes one more `attempt` and journals the return only when that call answers something other than `accepted`, with that call's answer as the reason, which is the last transient answer.
A receiver that stored the message but whose `accepted` was lost answers `accepted` with its held id through D3 step 3, and the entry is journaled delivered, not returned.

`attempt` builds a `NewClient` over `hubs.Geometry(hubPath)` with an empty claim, sends one `VerbDeliver` under a named 30-second timeout and reads `Delivery`.
The spawn it hands the client calls `hubs.Spawn` and counts one start for the hub, and past `maxHubStarts`, three, the client gets a nil spawn, so it only reconnects.
The count sits in the worker's scheduler-owned map, since a worker can retire and start again, resets only when that hub's daemon answers a call, and a restart of the sending daemon empties it.
`Client.exchange` returns a daemon's refusal, plain or coded, as `daemonRefusal`, which keeps the same `Error` text, so `attempt` tells an answering daemon from a lost connection.
A coded refusal's `daemonRefusal` wraps its `codedRefusal`, so `errors.Is` against `ErrStrandClaimOutsidePane` and `ErrProgramClaimInStrandPane` holds for every existing caller.
`connLostError` and a failed spawn give the code `connect-failed`, `ErrWaitTimeout` the code `timeout`, a failed geometry the code `hub-unreadable`, and a connect past the cap the code `start-capped`, with one Warn per episode naming the hub's daemon log.
Every other `Client.exchange` failure, the hub-path mismatch and undecodable JSON included, gives the code `bad-answer`, with no socket text in the reason.
An answer with neither `accepted` nor `refused` also gives `bad-answer`, with none of its text in the reason.
Text from the other side is quoted through `quoteAnswer`, whatever the socket that wrote it, so a `daemonRefusal`'s text, coded or not, and a `refused` answer's `Reason` reach the notice only quoted.

`settleReturned` reads the strand listing before the lock, journals `eventOutboundReturned` with the reason, answers `OutcomeReturned` with `outboundEntry.message()`, and posts a `<shortname>:CREEL` notice to the sender naming the receiver, the reason and `lym creel log --id <id>`, under `noticeUnansweredLocked`'s rule: a strand no longer listed gets none, and a notice that fails to store is logged and undoes nothing.
`quoteAnswer` turns line breaks into spaces, cuts on a rune boundary to `returnReasonMaxBytes`, 200, and wraps the text in double quotes with its quote marks and backslashes escaped.
A reason creel wrote as a code is not quoted.

A delivered message that expects a reply stays in `outbound`, held by no local strand, so no sweep settles it, and retention does not carry it.

`Scheduler.Stop` waits on the worker through its `WaitGroup`, so a call in flight ends before the daemon does.
Bound: that stop can take the connect wait after a start plus the 30-second call timeout.
`PATTERN-daemon-wakeups`: `unboundedWaitFiles` gains `internal/creelengine/outbound.go`, and the background's list gains a bullet for the worker, with its one-second base, its one-minute ceiling and the hour that ends it.
`PATTERN-child-processes`: `spawnDaemon` logs and reaps each start, `TestHubDirectory` pins that `Spawn` goes through it, and the count is capped here.

`TestScheduler_Outbound` is tier0 over `newHubPair`, each row with its own pair, calling `unpark` and moving the rigs' fake clocks; a `listenHubFake` stands in for a receiver whose answer a row scripts.
Rows:
- A post reaches the receiver's pool and the log shows its remote delivery.
- A post while the receiving daemon is down starts it through `creelfake.Hubs`'s spawn and is delivered.
- A refusal returns at once, sends one `CREEL` notice and ends a `--wait-reply` wait as returned; the notice is not sent to a sender no longer listed.
- A receiver down for the hour returns at the hour by the fake clock with the last transient reason.
- A pending outbound line written at `epoch` minus 40 minutes before `startSending` returns 20 minutes after it, so the hour counts across a restart.
- The retry waits read one second, doubling to a minute, capped at what is left of the hour, and a new post during a wait moves neither that wait nor the count of calls.
- Two posts to a held receiver arrive oldest first.
- A `deliver` a `hubFake` holds across the hour ends before the return is journaled: no return stands until the row releases the answer.
- The final call at the hour answering `refused` returns with that reason.
- A pending outbound line past the hour, beside a receiving journal holding its inbound record, is journaled delivered with the held id, not returned.
- A pending message survives a sending daemon restart and is delivered after it.
- A receiving daemon that exits at each start is started three times across the hour by `Hubs.Spawns`, with new posts in between, and the message still returns at the hour with `start-capped`.
- A daemon that answers a call resets the count, so a later exit is started again; a sending daemon restart empties it.
- An answer of neither word and an unknown-verb refusal are retried, not returned at once; at the hour the notice gives `bad-answer` for the neither-word answer and the quoted text for the unknown-verb refusal.
- A `hubFake` that never answers gives the `timeout` code after 30 seconds of the fake clock, a geometry error `hub-unreadable`, and a failing spawn `connect-failed`, each retried.
- A `hubFake` answering `Delivery` `refused` with a `Reason` of over 200 bytes, holding a quote, a backslash, a line break and a multibyte rune across byte 200, returns at once, and the notice quotes it cut on a rune boundary, with the line break made a space and the quote and backslash escaped.
- A `hubFake` answering with a foreign `HubPath` holding a quote and a line break is retried, and at the hour the notice gives the reason `bad-answer` and holds none of that text.
- A `hubFake` writing a non-JSON line is retried, and at the hour the notice gives the reason `bad-answer` and holds none of that line.
- A `hubFake` answering `OK: false` with the `Code` `CodeStrandClaimOutsidePane` and an `Error` holding a quote and a line break is retried, and at the hour the notice quotes that text with the line break made a space and the quote escaped.
- An id clash answered by a real receiver reaches the notice whole, naming the held range and the retention.
- A delivered message that expects a reply is not settled by a sweep on the sending side: the row moves the clock past two sweep ceilings with one `Advance`, and the log holds no `unanswered` or `undelivered`.
- On a `newEnv` store with `SetHubs` and a `hubFake` answering `accepted`, a failing journal write leaves the entry pending, makes no second call until the one-second wait passes, and the retry settles it.

**ImpactSummary:** A new daemon goroutine per receiving hub, started only for a pending outbound message; `Client.exchange`'s refusal keeps its text and its sentinel for every existing caller, and `TestDaemonWakeups_UnboundedWaitsStayAboveTheFloor` scans the new file.
