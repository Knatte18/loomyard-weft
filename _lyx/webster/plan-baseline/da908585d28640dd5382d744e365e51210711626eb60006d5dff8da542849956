# Card 4 — spawn-exit-signal

**Edit:**
- `internal/creelengine#Client`
- `internal/creelengine#NewClient` -> `func NewClient(geom Geometry, claim Claim, spawn func() (Spawn, error), clock Clock) *Client`
- `internal/creelengine#Client.connect`
- `internal/creelengine_test#rig.client`
- `internal/creelengine_test#TestDaemon_Protocol`
- `internal/creelengine_test#TestDaemon_Lifecycle`
- `internal/creelcli#clientSeam`
- `internal/creelcli#seam`
- `internal/creelcli#runVerb`
- `internal/creelcli#spawnDaemon` -> `func spawnDaemon(hubPath, logPath string) (creelengine.Spawn, error)`
- `internal/creelcli#newRig`
- `internal/creelcli#TestCreelCLI`
- `internal/creelengine/doc.go`
- `internal/creelcli/doc.go`
- `pattern/PATTERN-daemon-wakeups.md`

**Create:**
- `internal/creelengine#Spawn`
- `internal/creelengine#SpawnExit`
- `internal/creelcli#reapDaemon`

**Uses:**
- `internal/creelengine#ConnectAttempts`
- `internal/creelengine_test#rig.spawn`
- `internal/creelengine_test#rig.options`
- `internal/creelengine_test#externalClaim`
- `internal/creelengine_test#result`
- `internal/creelengine_test#callAsync`

**Intent:** Fix #683 (D4): the client stops waiting once the daemon it started has exited.

`Spawn` is what a client's spawn seam answers: `PID`, the pid of the started daemon, and `Exited`, a receive-only channel of `SpawnExit` that fires once when that process exits, nil for a seam that gives no signal.
`SpawnExit` carries `LostRace`, true when the daemon exited because another daemon holds the hub's lock.
`NewClient` takes the seam in that shape, and `Client` keeps it.
`Client.connect` calls the seam once per connect, so a reconnect that spawns again waits on its own spawn's signal.
Between connects it waits on the backoff timer or the signal, whichever comes first.
On an exit with `LostRace` it stops reading the signal and keeps waiting on the same timer with today's backoff, since the winner takes the lock before it folds the journal and binds the socket and may not answer yet.
On any other exit it dials once more at once; that connect's success returns the connection, and its failure returns the existing error naming the socket and quoting the daemon log for the spawn's pid.
Once a connect has succeeded the signal is not read again, so a daemon that dies later affects the call only through its dropped connection.
A nil `Exited` keeps today's waits.
The attempt cap and the backoff ceiling stay as they are.
Bound: the signal ends the wait early only on an exit other than a lost start race, when the spawned daemon cannot answer; a start that fails before it tries the lock ends the wait even while another client's daemon is starting, and that verb returns the daemon-log error unless its one last connect reaches that daemon.

In `internal/creelcli`, `clientSeam.spawn` answers a `creelengine.Spawn`, `runVerb` hands it to `NewClient`, and `spawnDaemon` returns the started pid with the channel `reapDaemon` gives.
`reapDaemon` waits on the started command in the background, logs the exit with the pid and the exit status at Info, as `PATTERN-spawn-observability` asks of a wait, and sends one `SpawnExit` whose `LostRace` is true exactly when the exit status is 0: `lyx creel daemon` exits 0 when another daemon holds the lock or it stops cleanly, and 1 when its start fails.
Under go test `spawnDaemon` still starts nothing.

Tests in `TestDaemon_Lifecycle`, on a client clock the test moves by hand:
- "ClientWhoseSpawnedDaemonExitsAtOnceReturnsTheLogErrorWithoutTheBackoff": the seam answers pid 4242 and an already fired non-race exit with no daemon serving; the call returns the error naming the socket and the daemon log while the clock never advances.
- "ClientWhoseSpawnedDaemonExitsMidWaitConnectsOnceMoreToADaemonThatServes": the seam answers an unfired channel; once the client sleeps, the test starts a daemon over the rig and fires a non-race exit, and the call succeeds with no clock move.
- "ClientWhoseSpawnLostTheStartRaceKeepsTheBackoffAndConnects": the seam fires a lost-race exit at once; the client still sleeps, the test then starts a daemon that answers, advances the clock one second, and the call succeeds, which covers a winner that holds the lock but does not yet answer at the exit.
- "WaitingCallThatReconnectsWaitsOnlyOnItsOwnSpawnsSignal": an `inject --wait` connects to a daemon the first spawn started, whose channel is unfired; the test stops that daemon and then fires the first spawn's channel with a non-race exit, which the established call never reads; the reconnect's spawn answers a second, unfired channel, the client sleeps its backoff instead of ending on the first spawn's exit, and once the test restarts a daemon and advances the client clock one second the call ends with its injected outcome.
The existing row "ClientWithNoListenerSpawnsOnceAndGivesUpNamingTheSocketAndQuotingTheDaemonLog" answers a `Spawn` with no channel and keeps pinning `ConnectAttempts-1` sleeps from one second, the no-signal case; it also runs one variant whose `Spawn` fires a lost-race exit at once, which with no daemon serving still sleeps `ConnectAttempts-1` times and returns the same error naming the socket and the daemon log; `rig.client` adapts its spawn the same way.
`TestCreelCLI` gains the row "the spawned daemon's reap logs its exit and reads exit 0 as a lost start race", the one test that pins the real subprocess boundary of the reap, since `spawnDaemon` starts nothing under go test: under `logcapture.CaptureVerbose`, `reapDaemon` over a started `sh -c 'exit 0'` sends `LostRace` true, over `sh -c 'exit 1'` sends false, and over `sh -c 'kill -9 $$'`, a wait error that is no exit code, sends false, and the captured Info log holds each pid and status.
`newRig`'s seam stub answers a `Spawn` with no channel.

Docs in the same commit: the Daemon section of `internal/creelengine/doc.go` says the client stops waiting at an exit of its spawned daemon other than a lost start race, after one last connect; `internal/creelcli/doc.go` says the spawn reaps the detached daemon in the background, logs its exit and reads exit 0 as a lost start race; the creel waiting-verbs line of `pattern/PATTERN-daemon-wakeups.md` adds that the client's connect wait after a spawn also ends on that exit.

**ImpactSummary:** `creelengine.NewClient`'s spawn parameter changes shape; its only production caller is `internal/creelcli`'s `runVerb`, and its test callers are the creelengine rig and two `TestDaemon_*` rows, all listed here.
