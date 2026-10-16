# Card 12 — daemon-log-file

**Create:**
- `plan:internal/creelengine#DaemonStartHeader` -> `func DaemonStartHeader(pid int, at time.Time) string`
- `plan:internal/creelengine#DaemonLogPrefix` -> `func DaemonLogPrefix(pid int) string`
- `plan:internal/creelengine#daemonLogExcerpt` -> `func daemonLogExcerpt(path string, pid int) string`
- `plan:internal/creelcli#openDaemonLog` -> `func openDaemonLog(path string) (*os.File, error)`
- `plan:internal/creelcli#daemonLogCap` -> `const daemonLogCap = 1 << 20`

**Edit:**
- `internal/creelengine#Geometry`
- `internal/creelengine#Client`
- `internal/creelengine#NewClient` -> `func NewClient(geom Geometry, claim Claim, spawn func() (int, error), clock Clock) *Client`
- `internal/creelengine#Client.connect`
- `internal/creelengine/doc.go`
- `internal/creelengine_test#rig.client`
- `internal/creelengine_test#TestDaemon_Protocol`
- `internal/creelengine_test#TestDaemon_Lifecycle`
- `internal/hubgeom#CreelGeometry`
- `internal/hubgeom#TestCreelGeometry`
- `internal/creelcli#spawnDaemon` -> `func spawnDaemon(hubPath, logPath string) (int, error)`
- `internal/creelcli#clientSeam`
- `internal/creelcli#seam`
- `internal/creelcli#runVerb`
- `internal/creelcli#newRig`
- `internal/creelcli#TestCreelCLI`
- `internal/creelcli/doc.go`

**Uses:**
- `internal/creelengine_test#rig`
- `internal/creelengine_test#externalClaim`
- `internal/testkit/creelfake#NewClock`
- `internal/testkit/creelfake#Clock.AutoAdvance`
- `internal/creelcli#runCreel`
- `internal/hibernatecli#spawnDaemon`
- `internal/testkit/logcapture#Capture`

**Intent:** Decision D5, client half: the detached daemon's stderr goes to a per-hub log file, and a client that gives up connecting quotes that start's lines from it.

Geometry: `creelengine.Geometry` gains `DaemonLogPath`, the daemon log file in creel's state directory beside the journal, and `hubgeom.CreelGeometry` sets it to `<StateDir>/daemon.log`, under the board worktree's never-tracked `.lyx` (`PATTERN-durable-vs-ephemeral-state`); neither the spawn nor the daemon verb builds the path (`PATTERN-told-geometry`, `PATTERN-lyxdirs-single-declarer`).
`TestCreelGeometry`'s expected geometry gains the field.

The line format, declared once in `creelengine` and used by both ends: `DaemonStartHeader(pid, at)` is the start header line `creel daemon start pid=<pid> at=<RFC 3339 UTC>`, and `DaemonLogPrefix(pid)` is `pid=<pid> `, which opens every line the daemon writes through its logger (card 13).

Spawn: `spawnDaemon(hubPath, logPath)` takes the daemon's stderr from `openDaemonLog` and points the detached process's stderr at it, keeping stdin and stdout null, and returns the started process's pid, in the shape `internal/hibernatecli`'s `spawnDaemon` already gives its detached daemon and its log.
`openDaemonLog` truncates the file first when it is larger than `daemonLogCap` (1 MiB), then opens it for append, creating it and its directory.
A log that cannot be opened does not stop the spawn: `openDaemonLog` logs it at Warn and answers a nil file with its error, and `spawnDaemon` then leaves the daemon's stderr null, so the fallback is decided in `openDaemonLog`, which a test reaches, not behind `spawnDaemon`'s `go test` guard.
Under `go test` `spawnDaemon` still starts nothing and returns its error.
`clientSeam.spawn` takes the hub and log paths and answers the pid, `seam` keeps `spawnDaemon`, and `runVerb` hands `NewClient` a spawn closure over `geometry.DaemonLogPath`.

Client: `NewClient`'s spawn answers the pid it started, and `Client.connect`, when it gives up after `ConnectAttempts`, appends `daemonLogExcerpt(geom.DaemonLogPath, pid)` to its error, which keeps naming the socket and now names the log file.
`daemonLogExcerpt` reads the file and quotes the lines that open with `DaemonLogPrefix(pid)`, and the lines carrying no `pid=` prefix (a runtime panic trace) that follow this pid's own start header with no other start header after it.
When another start header follows this pid's own, it quotes only the pid-tagged lines and says the file also holds lines it cannot attribute.
A file with no such line says the file holds no line from that start; a file that cannot be read says the daemon log could not be opened, which is also what a spawn that could not open it leaves behind.
A truncation that erased this start's header but not its later lines still finds those lines by pid.
Bound: the excerpt reads one capped file once, at a give-up only.

Callers of `NewClient`: `runVerb`, and in tests `rig.client`, which keeps its `func() error` parameter and adapts it (nil stays nil), and the one direct call in `TestDaemon_Protocol`.
`newRig` in `cli_test.go` sets the seam's spawn to one that starts nothing and answers pid 0.

Tests:
`TestDaemon_Lifecycle`'s `ClientWithNoListenerSpawnsOnceAndGivesUpNamingTheSocket` keeps its socket, spawn-count and sleep assertions and becomes a table over a log file it writes into the rig's state directory; each row sets `r.geom.DaemonLogPath` to that file and calls `creelengine.NewClient` directly with a spawn answering pid 4242, since `rig.client`'s adapted spawn answers pid 0:
lines of pid 4242 and a pid-less panic line after its own header are quoted, and pid 77's lines and header and a pid-less line before 4242's header are not;
another start header after 4242's own leaves only 4242's tagged lines quoted and the unattributed-lines note;
4242's tagged lines with no 4242 start header, as a truncation leaves them, are still quoted;
a file holding only pid 77's lines says it holds no line from that start;
and an absent file says the log could not be opened; every row's error names the log file.
`TestCreelCLI` gains a row where `openDaemonLog` appends to a small existing log, truncates one over the cap before appending, and on a path whose parent is a regular file answers a nil file and an error and logs a Warn, the fallback that leaves the daemon's stderr null.
`openDaemonLog` is unexported and tested in the package, because `spawnDaemon` starts nothing under `go test`, so no verb reaches the log's opening.

`internal/creelengine/doc.go`'s Daemon paragraph and `internal/creelcli/doc.go` name the daemon log and what a give-up quotes.

**ImpactSummary:** Re-signs NewClient and spawnDaemon with every caller listed here (runVerb, seam, rig.client, TestDaemon_Protocol, newRig); the geometry gains one field read by the spawn and the give-up only.
