# Card 7 — creel-stall-wait

**Create:**
- `plan:internal/creelengine#StallNote` -> `type StallNote struct`
- `plan:internal/creelengine#Client.CallWatching` -> `func (c *Client) CallWatching(req Request, timeout time.Duration, handle func(Response) error, progress func(Response)) (Response, error)`
- `plan:internal/creelcli#runCreelSplit` -> `func runCreelSplit(cwd string, stderr io.Writer, args ...string) cliRun`
- `plan:internal/creelcli#lockedBuffer` -> `type lockedBuffer struct`
- `plan:internal/creelcli#lockedBuffer.Write` -> `func (b *lockedBuffer) Write(p []byte) (int, error)`
- `plan:internal/creelcli#lockedBuffer.String` -> `func (b *lockedBuffer) String() string`

**Edit:**
- `internal/creelengine#Waiter`
- `internal/creelengine#Outcome`
- `internal/creelengine#Response`
- `plan:internal/creelengine#Store.noteStall`
- `internal/creelengine#Store.addWaiterLocked`
- `internal/creelengine#Store.settleInjectLocked`
- `internal/creelengine#daemon.awaitOutcome`
- `internal/creelengine#Client.Call`
- `internal/creelengine#Client.exchange` -> `func (c *Client) exchange(conn net.Conn, req Request, timer <-chan time.Time, state *waitState, handle func(Response) error, progress func(Response)) (Response, error)`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#TestDaemon_Protocol`
- `internal/creelengine_test#TestScheduler_Injects`
- `internal/creelcli#injectCmd`
- `internal/creelcli#runVerb`
- `internal/creelcli#TestCreelCLI`

**Uses:**
- `plan:internal/creelengine#Stall`
- `plan:internal/creelengine#Stall.Reason`
- `plan:internal/testkit/creelfake#Delivery.SetStall`
- `internal/creelengine_test#rig`
- `internal/creelengine_test#journalText`
- `internal/creelengine_test#callAsync`
- `internal/creelcli#cliRun`
- `internal/creelcli#rig.startDaemon`
- `internal/clihelp#RunRootCtx`

**Intent:** Decision D2, waiting half: a waiting `inject --wait` hears each change of its receiver's stall reason as a progress line on stderr, and the one stdout envelope of an expired or timed-out wait carries the last reason and its time.

Engine: `StallNote` is a stall and the time it was read.
`Waiter` gains `Progress <-chan StallNote`, a one-slot channel where a newer note replaces an unread older one; only a wait on an inject gets one.
`Store.noteStall`, on a change it journals, sends the note to the waiters of every inject still queued for that receiver; `Store.addWaiterLocked` hands a new inject waiter the receiver's current note at once when one stands, so a wait resumed after a reconnect shows the reason too.
`Store.settleInjectLocked` writes the receiver's current stall as the `reason` of an `expired` record and answers the waiters with `Outcome` carrying it, as `Outcome`'s new `Stall` and `StallAt` fields; an inject that expires with no stall standing carries none.
An older binary ignores the `reason` key (`PATTERN-tolerant-records`).

Protocol: `Response` gains `Progress bool` (`json:"progress,omitempty"`), `Stall` (`json:"stall,omitempty"`) and `StallAt` (`json:"stall_at,omitzero"`).
`daemon.awaitOutcome` also waits on the waiter's progress channel and writes each note as a progress response carrying the inject id, `Stall` and `StallAt`, then keeps waiting; the final expired response carries the outcome's `Stall` and `StallAt`.
`Client.CallWatching` is `Client.Call` with a progress callback: `Client.exchange` hands every progress response to it and reads on, a nil callback ignoring them, and `Client.Call` becomes `CallWatching` with a nil callback, so its other callers see nothing new.
Bound: one progress line per change of reason per receiver, the same bound as the `stalled` record.

CLI: `injectCmd` with `--wait` calls `CallWatching`, writes each progress response as one line to `cmd.ErrOrStderr()` as it arrives (`creel: <receiver> is stalled: <Stall.Reason()> (at <RFC 3339 time>)`), never to stdout, and keeps the last one.
An expired outcome adds `stall`, `stall_reason` and `stall_at` to its envelope fields when the response carries a stall.
A wait the client times out on returns the last progress note's same three fields beside `ErrWaitTimeout`, and `runVerb`'s timeout branch prints the body's fields in its one error envelope, so a timeout with no note prints exactly as today.

Tests:
`TestDaemon_Protocol` gains a row where an external `CallWatching` inject with a 60 s TTL into a strand scripted `StallDraft` and then `StallTooShort` receives exactly two progress responses with those stalls in order, and ends expired with `Stall` too-short, and the journal's `expired` line carries `"reason":"too-short"`;
a row where, once a stall stands for a queued inject, a second `CallWatching` with `VerbWaitInject` on its id receives a progress response with that stall at once, as a wait resumed after a reconnect does;
and its existing `InjectWaitReportsExpired` row also asserts the expired response carries no `Stall`, since no stall stood.
The rows build their clients through `rig.client`, whose `NewClient` call card 12 later adapts.
`TestScheduler_Injects` gains a row over `Store.InjectWait` where two stall changes pass with the waiter's `Progress` unread, and the one read then gets the newer note only, which pins the one-slot replacement.
`runCreelSplit` runs the creel command as `RunCLIIn` does but writes stderr to the writer it is handed, through `clihelp.RunRootCtx` over a cwd-seeded context, and returns the stdout run; `lockedBuffer` is a mutex-guarded buffer a row hands it and reads while the verb runs in a goroutine.
`TestCreelCLI` gains rows, each running the verb in a goroutine through `runCreelSplit` and polling its `lockedBuffer` under a real-time deadline until the progress line arrived before it moves a clock:
over a strand the fake delivery reads `StallDraft`, `inject --wait --ttl 1m` exits 3 once the daemon clock passes the TTL, with one stdout envelope carrying `stall` `draft`, its `stall_reason` and a `stall_at`, while stderr holds the progress line naming the draft and stdout holds no progress text;
and over a strand read `StallTooShort`, `inject --wait --timeout 5s` exits 4 once the client clock passes the timeout, with the same three fields in its one envelope and a `stall_reason` naming `lyx reed attach`, the way forward the too-short reason gives.

`internal/creelengine/doc.go`'s Inject paragraph and protocol paragraph say a waiting inject gets progress lines and its expired answer carries the reason, and the `expired` record names it.

**ImpactSummary:** Extends the waiting-inject path in store, daemon, client and the inject verb; Call keeps its signature, so its callers and the other waiting verbs print as before.
