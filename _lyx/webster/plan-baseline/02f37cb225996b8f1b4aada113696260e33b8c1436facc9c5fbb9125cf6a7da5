# Card 10 — creelengine-daemon

**Create:**
- `internal/creelengine/protocol.go`
- `internal/creelengine/server.go`
- `internal/creelengine/client.go`
- `internal/creelengine/server_test.go`
- `internal/creelengine#Options`
- `internal/creelengine#Run`
- `internal/creelengine#Request`
- `internal/creelengine#Response`
- `internal/creelengine#Client`
- `internal/creelengine#NewClient`

**Edit:**
- `cmd/lyx#unboundedWaitFiles`
- `//cmd/lyx/testdata/test-budget.yaml`
- `//pattern/PATTERN-lyx-temp-root.md`
- `//pattern/PATTERN-daemon-wakeups.md`

**Intent:** Run the per-hub daemon and its socket, and give callers a Go client (D2, D3, D12's waiting half).
`Options` carries the `Geometry`, the `Delivery`, the `Clock`, the parent-pid function, the GOOS and a `Replaced func() bool` binary-identity probe (the CLI passes `selfexec.Watcher.Replaced`).
`Run` takes the lock at `LockPath` with `lock.TryAcquireWriteLock`; a second daemon that cannot take it returns `ErrDaemonRunning`, an error saying another daemon holds the hub.
Holding the lock, it removes a socket file no listener answers (dial, re-stat the same file, remove; the model is `reedengine`'s stale-socket removal), binds `SocketPath`, rebuilds the pool from the journal including queued injects with their consumed TTL, and starts the scheduler and sweep (card 9).
It never exits on idle; its own poll, once a second on the told clock, asks `Replaced` and the clock's UTC day change, which runs retention (card 6); on a replaced binary it starts no new send, waits for sends in flight to return and journals their outcomes, closes its listener, releases the lock and returns, so the next verb starts the new binary's daemon.
`protocol.go`: each connection carries one JSON-lines `Request` (verb, arguments, `Claim`) and its `Response`; every `Response` carries `HubPath`; a read's response is followed by the client's ack line on the same connection, and a connection that closes before the ack releases the reservation; a waiting request (`post --wait-reply`, `inject --wait`, `read --wait`) binds its connection to card 7's waiter registry on its message id, inject id or holder and keeps the connection open until its answer.
A request line that does not decode, or names a verb the server does not know, is answered with an error response naming the line's fault, and the connection closes; the pool is untouched.
The server resolves the sender of each request (card 8) from the peer pid before dispatching to the `Store` verb; a request that changes the pool is answered only after its journal line is synced.
`read --wait` blocks until the caller holds unread mail, then answers as `read` does.
`Client` dials `SocketPath`; when it cannot connect it calls the told `spawn` once and retries the connect up to a capped count, sleeping on the told clock from one second between tries, then fails naming the socket; it refuses a response whose `HubPath` is not its geometry's hub.
A waiting call takes a timeout (zero waits until an outcome), measured on the told clock; its expiry returns `ErrWaitTimeout`, which the CLI exits non-zero on; a dropped connection reconnects, starting the daemon through `spawn` if needed, and waits again on the same id or holder up to a capped count, after which it returns an error naming the socket and the reconnects spent.
A reconnecting waiter whose outcome happened while it was disconnected is answered at once from the pool: an undelivered reply to its message id is handed over and journaled `delivered`, a message returned undelivered is reported so, and an inject's injected, unconfirmed or expired outcome is reported.
Bound: the waiting verbs' reconnects and the start retries are capped by count as well as time (`PATTERN-spawn-observability`).
`internal/creelengine/server.go` and `internal/creelengine/client.go` join `cmd/lyx#unboundedWaitFiles`, and `pattern/PATTERN-daemon-wakeups.md`'s What-counts list gains creel's daemon poll (binary identity and day change, once a second) and its waiting verbs (`post --wait-reply`, `inject --wait`, `read --wait`, which wait on their connection and reconnect from one second); no sub-second literal is needed in either file.
`pattern/PATTERN-lyx-temp-root.md` gains a line: creel's per-hub socket `creel-<shortname>-<h>.sock` is a long-lived file directly in the lyxtemp root beside the run directories, and `Root.Sweep` touches no unprefixed entry, so it never removes it.
`server_test.go` is a `package creelengine_test` file using exported API only, since an in-package test importing `creelfake` is an import cycle.
`server_test.go`, every daemon in process on a socket under the test's lyxtemp run directory with `creelfake`'s delivery and clock.
The test process is the peer of every in-process client, so a row that calls as a strand scripts a `creelfake` strand whose `PanePID` is `os.Getpid()`, which card 8's walk resolves to that strand, and sets the claim's `LYX_STRAND_NAME` to that strand's name, which agrees with the ancestry on Linux and is the identity off Linux; a row that calls as EXTERNAL scripts none, and its receiver strands rely on card 7's default `creelfake` pane pid, since a live strand with pane pid 0 makes card 8 refuse a peer in no known pane and a pid in the test's ancestry makes the peer that strand.
`TestDaemon_Protocol` (post acknowledged with its id after its line is in the journal file; as a strand, read with ack marks read and a client dying before the ack leaves the messages unread; `read --wait` returning once a post arrives for the caller; `read --wait`, `post --wait-reply` and `inject --wait` returning `ErrWaitTimeout` once the fake clock passes their timeout; a reply reaching a waiting EXTERNAL `post --wait-reply`; `inject --wait` reporting injected, unconfirmed and expired; a `post --wait-reply` whose message returns undelivered answered so; a client refusing a daemon answering for another hub path; a malformed line and an unknown verb each answered with an error and no journal line; the sender of a request resolved from the peer, a `LYX_STRAND_NAME` claim naming another strand refused, that row skipped when `runtime.GOOS` is not `linux` with a reason naming D4's Linux-only ancestry check).
`TestDaemon_Lifecycle` (two daemons started at once on one hub: one serves and the other returns `ErrDaemonRunning`; two concurrent first client calls whose spawns both start a daemon end with one serving and both calls answered; a client with no listener calls `spawn` once and retries the connect up to the cap, then fails naming the socket; a stale socket file with no listener is removed and rebound; a journal left as a killed daemon leaves it, its last `aged` event a minute old and no stop-time record, rebuilds on a new daemon with each queued inject's consumed TTL at most a minute short; a journal holding `sending` without outcome yields unconfirmed on restart and nothing typed again; a waiter connected across a restart reconnects and receives the injected outcome, and in another row the returned-undelivered outcome, journaled while it was away; an EXTERNAL `post --wait-reply` that loses its connection, a strand replying while no waiter is connected (journaled `undelivered`), and the reconnected waiter receiving the reply at once with the journal holding a `delivered` event for it; a waiter whose daemon never returns gives up after the capped reconnects naming the socket; the fake clock crossing a UTC day boundary runs retention from the poll; `Replaced` turning true lets the in-flight send finish and journal, then `Run` returns and the lock is free).
`cmd/lyx/testdata/test-budget.yaml` raises `internal/creelengine` by two.

**ImpactSummary:** New daemon, protocol and client files in creelengine, two files added to the daemon-wakeups scan list, two pattern background edits and one budget raise; no existing caller.
