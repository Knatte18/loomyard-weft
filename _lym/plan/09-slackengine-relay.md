# Card 9 — slackengine-relay

**Create:**
- `plan:internal/slackengine#RelayDeps` -> `type RelayDeps struct`
- `plan:internal/slackengine#RunRelay` -> `func RunRelay(ctx context.Context, geom Geometry, deps RelayDeps) error`
- `plan:internal/slackengine#ErrRelayInStrandPane` -> `var ErrRelayInStrandPane = errors.New("slack: the relay runs in a strand pane that is no orch's")`
- `plan:internal/slackengine#StartOutcome` -> `type StartOutcome string`
- `plan:internal/slackengine#EnsureRelay` -> `func EnsureRelay(geom Geometry) (StartOutcome, error)`
- `plan:internal/slackengine#EnsureRelayWhileAway` -> `func EnsureRelayWhileAway(geom Geometry) (StartOutcome, error)`
- `plan:internal/slackengine#RelayLive` -> `func RelayLive(geom Geometry) (bool, error)`
- `plan:internal/slackengine#StopRelay` -> `func StopRelay(geom Geometry, callerPID int) ([]runproc.Ended, error)`
- `plan:internal/slackengine#SpawnRelayFromIntermediate` -> `func SpawnRelayFromIntermediate(geom Geometry) error`
- `plan:internal/slackengine#AwaitReparent` -> `func AwaitReparent(intermediate int, parentPID func() int, clock Clock, bound time.Duration) error`
- `plan:internal/slackengine#AwaitFirstRead` -> `func AwaitFirstRead(geom Geometry, clock Clock, bound time.Duration) error`
- `plan:internal/slackengine#ErrSpawnUnderTest` -> `var ErrSpawnUnderTest = errors.New("slackengine: no relay spawned under test")`
- `plan:internal/slackengine#starter` -> `type starter struct`
- `plan:internal/slackengine#starter.ensure` -> `func (s starter) ensure(geom Geometry, whileAway bool) (StartOutcome, error)`
- `plan:internal/slackengine#starter.spawnRelay` -> `func (s starter) spawnRelay(geom Geometry) error`
- `plan:internal/slackengine#errFirstReadAccepted` -> `var errFirstReadAccepted = errors.New("slack: the first read was accepted and its mail left for the outbound read")`
- `internal/slackengine/relay_test.go`
- `plan:internal/slackengine#TestRunRelay` -> `func TestRunRelay(t *testing.T)`
- `plan:internal/slackengine#TestEnsureRelay` -> `func TestEnsureRelay(t *testing.T)`
- `plan:internal/slackengine#TestRelayWaits` -> `func TestRelayWaits(t *testing.T)`

**Edit:**
- `cmd/lym#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`
- `pattern/PATTERN-child-processes.md`

**Extends:**
- `internal/slackengine/doc.go`

**Uses:**
- `plan:internal/slackengine#inbound.poll`
- `plan:internal/slackengine#outbound.read`
- `plan:internal/creelengine#ErrProgramClaimInStrandPane`
- `plan:internal/runproc#RoleSlackRelay`
- `internal/runproc#EndRole`
- `internal/runproc#SelfRecord`
- `internal/runproc#Register`
- `internal/lock#TryAcquireWriteLock`
- `internal/lock#AcquireWriteLockWithin`
- `internal/lock#ProbeGrace`
- `internal/proc#ReapInheritedChildren`
- `internal/selfexec#New`

**Intent:** Add the relay process's life: its lock, first read, loops, re-exec and log (D3), its double-fork start, the starts the record names, its stop, and the waits `lym slack away` and the relay itself make (D3, D6).

`RunRelay` (in `relay.go`), told `RelayDeps{Mailbox, Slack, Hub, Clock, PID func() int, ReexecLockWait time.Duration, CheckBinary func(beforeExec, failed func()) error, ReapInherited func() error, Register func() error}`.
Production wiring (card 10) fills every field: `PID` is `os.Getpid`, `ReexecLockWait` 10 seconds, `CheckBinary` `selfexec.New().CheckWith`; a test sets every field, and a zero `ReexecLockWait` waits only `lock.ProbeGrace`.
1. Refuses through `RequireConfigured` with no token or setup file, naming `lym slack login` or `lym slack setup` (D3).
2. Calls `ReapInherited` (`proc.ReapInheritedChildren`) before it starts anything, then takes `LockPath`: a relay whose `RelayState.PID` equals `deps.PID()` is a re-exec and waits up to `ReexecLockWait` for the lock its old image released at the exec; any other start waits `lock.ProbeGrace`, as dobby's fresh start does, so a liveness probe (`RelayLive`, which takes the lock briefly) never makes a starting relay exit. A relay that still cannot take the lock exits with no error before its first poll, so a second start for the hub starts nothing and two relays never both poll.
3. Calls `Register` (`runproc.Register` under `runproc.HubOwner` with `runproc.SelfRecord(runproc.RoleSlackRelay)`) and writes `RelayState` afresh with its pid and start time, its `FirstReadAt` zero.
4. Makes its first creel read as `<shortname>:SLACK` before any Slack call: `Mailbox.ReadAll` with a handler that returns the package sentinel `errFirstReadAccepted`, so any mail stays unread for `outbound.read`; a read with no mail calls no handler and returns nil. Either nil or an error matching `errFirstReadAccepted` is an accepted read and writes `FirstReadAt`.
   A refusal matching `creelengine.ErrProgramClaimInStrandPane`, this run's or the first after a re-exec, writes `FirstReadRefusal`, releases the lock and returns `ErrRelayInStrandPane` wrapping creel's message, with the way forward "start the relay with `lym slack start` from the operator's terminal or an orch's pane", so a relay run by hand from a non-orch strand's pane polls Slack for nothing and leaves the lock free.
   Any other refusal or failure, such as an unknown live pane process or a daemon that is down, writes `FirstReadRefusal`, is retried with the poll's backoff while the relay holds its lock and calls no Slack method, and the first accepted read clears the refusal.
5. Runs `outbound.read` in a goroutine and `inbound.poll` in the loop, each waiting its own backoff on the told clock: a 5-second floor, doubled after each empty pass up to 60 seconds, back to the floor on activity (a forwarded or answered message, a successful outbound post, or a presence change the poll reads). A `*RateLimitError` from either waits its `RetryAfter` and doubles the shared floor, up to 60 seconds, for the rest of the run (D4).
   Each successful Slack call writes `LastSlackOK`.
6. At each poll calls `CheckBinary` with a `beforeExec` that releases the lock, so a replaced binary re-execs the relay, which runs from step 2 again (D3), and a `failed` that retakes the lock with `lock.TryAcquireWriteLock`; when the lock is taken by then, as by a relay a start launched in the window, the old image logs it and exits, so only one relay polls.
7. Appends every refused or failed read and every Slack failure to `LogPath`, Warn once per episode in the process log, as `internal/logger`'s Level policy asks.

The start (D3), in `start.go`:
- `RelayLive` probes the lock: held means live.
- `EnsureRelay` and `EnsureRelayWhileAway` call `starter.ensure` over the production `starter`, whose `spawn` field runs `<LymExecutable> slack relay --hub-path <hub> --fork` from the hub directory, detached with `proc.Detach` in a new session, logs the spawn at Info and waits on that intermediate with `cmd.Wait`; under `go test` it returns `ErrSpawnUnderTest`. Tests build a `starter` with a fake `spawn` and `clock` and call `ensure` and `spawnRelay` on it, so every test keeps `t.Parallel`.
- `starter.ensure` answers `not-configured` with no error when the token or the setup file is missing, `live` when the relay is live, and otherwise starts it and answers `started`. With `whileAway`, it answers `terminal` and starts nothing unless `ReadPresence` reads `slack`, for the prime dobby's tick (card 12).
- The tick's restarts are capped (`PATTERN-child-processes`, a retry loop around a spawn caps its count): `RelayState` carries `StartFailures` and `NextStartAt`. A `whileAway` start that finds the relay down while the last relay it started never wrote `FirstReadAt` counts one failure, and waits from one minute, doubling up to 30 minutes, before it spawns again; at the fifth failure in a row it spawns no more, answers `gave-up`, and posts nothing itself: its caller logs it at Warn once. Any relay's accepted first read resets both fields, and `EnsureRelay` without `whileAway`, the operator's or the bring-up's start, ignores the cap and resets it.
- `SpawnRelayFromIntermediate` is the intermediate's step, `starter.spawnRelay` over the production spawn: it starts `<LymExecutable> slack relay --hub-path <hub> --after <its own pid>` detached, stdout and stderr on `LogPath`, logs the spawn and exits without waiting on it, the one release `PATTERN-child-processes` admits (card 1), so the relay is reparented out of the starter's tree, and out of any pane's tree, whoever starts it.
- `AwaitReparent` is the relay's wait before step 4 when started with `--after`: it reads its parent once a second until it is no longer the intermediate, and errors after the bound (30 seconds) naming the intermediate; the relay verb then exits with that error before taking the lock (card 10).
- `StopRelay` ends the relay through `runproc.EndRole(geom.RunProcDir, runproc.HubOwner, runproc.RoleSlackRelay, callerPID)` (`PATTERN-child-processes`).
- `AwaitFirstRead` reads `RelayState` once a second until the relay it names is live (its pid and start time match a running process) and has written `FirstReadAt`, and refuses after the bound (30 seconds) naming `LogPath`. A state whose relay is gone but recorded `FirstReadRefusal` is refused at once naming creel's refusal, and a live relay's recorded refusal is named at the bound. Each refusal ends with the way forward "fix what it names, then run `lym slack away` again; the relay keeps its lock and retries" (D6).

`cmd/lym`'s `unboundedWaitFiles` adds `internal/slackengine/relay.go`; `pattern/PATTERN-daemon-wakeups.md`'s "What counts" list adds the relay's loops, from five seconds doubling to sixty; `pattern/PATTERN-child-processes.md`'s run-process-owner bullet names the Slack relay beside the detached board sync as a `runproc.HubOwner` process.

`internal/slackengine/relay_test.go` is tier0, with the in-memory fakes of `inbound_test.go`, a temp `Geometry`, a fake clock and distinct fake pids per run:
`TestRunRelay` rows — with no token, and with no setup file, it refuses naming `lym slack login` and `lym slack setup` and takes no lock; a first read accepted calls `Register` once, writes `FirstReadAt`, and only then does the fake Slack see a call; a first read with unread SLACK mail is accepted, polls, and leaves the mail unread for the outbound read; a first read refused with `ErrProgramClaimInStrandPane` returns `ErrRelayInStrandPane` naming `lym slack start`, makes no Slack call, and the lock is free for a later run of another pid, which takes it; a first read refused with creel's unknown-pane refusal keeps the lock, makes no Slack call, records the refusal, and after the fake mailbox accepts makes its first read and polls; a second `RunRelay` of another pid while the first holds the lock returns before any read; a run whose `RelayState.PID` equals its own pid takes a lock its old image releases within `ReexecLockWait`, while a fresh run against the same held lock returns; a lock held only for a probe's grace is taken by a fresh run; a `CheckBinary` that reports a replaced binary releases the lock in `beforeExec`, and one whose exec fails retakes it in `failed`, or exits when another holder took it; a successful Slack call writes `LastSlackOK`; the inbound wait doubles after empty polls up to 60 seconds, returns to 5 on activity, and a `429` waits its `Retry-After` and doubles the floor to 10.
`TestEnsureRelay` rows, over a `starter` with a fake spawn — a hub with no token or no setup answers `not-configured` and spawns nothing; a live relay answers `live` and spawns nothing; a down relay answers `started` with one spawn of `slack relay --hub-path <hub> --fork`; `spawnRelay` spawns `slack relay --hub-path <hub> --after <pid>` once and does not wait on it; `whileAway` spawns for a down relay only while presence is `slack`, and with presence `terminal` spawns nothing, the state `lym slack stop` leaves (D3's criterion on the next dobby tick); a `whileAway` start after a relay that never made its first read waits a minute before the next spawn, and after five such failures answers `gave-up` and spawns nothing until a start without `whileAway` resets the cap.
`TestRelayWaits` rows — `AwaitReparent` returns once the parent changes and errors at the bound; `AwaitFirstRead` returns on a live relay's `FirstReadAt`, ignores a state whose pid is gone when it records no refusal, refuses at once naming creel's refusal when a gone relay recorded one, refuses at the bound naming the log, and each refusal names `lym slack away`.

**ImpactSummary:** New process code inside the new package, plus the one declared wait-file list entry and two background-file lists naming the relay; no other caller until cards 10 to 13.
