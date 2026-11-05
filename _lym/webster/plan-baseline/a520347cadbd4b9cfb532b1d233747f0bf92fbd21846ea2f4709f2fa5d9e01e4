# Card 9 — slackengine-relay

**Create:**
- `plan:internal/slackengine#RelayDeps` -> `type RelayDeps struct`
- `plan:internal/slackengine#RunRelay` -> `func RunRelay(ctx context.Context, geom Geometry, deps RelayDeps) error`
- `plan:internal/slackengine#ErrRelayInStrandPane` -> `var ErrRelayInStrandPane = errors.New("slack: the relay runs in a strand pane that is no orch's")`
- `plan:internal/slackengine#StartOutcome` -> `type StartOutcome string`
- `plan:internal/slackengine#EnsureRelay` -> `func EnsureRelay(geom Geometry, primeName string) (StartOutcome, error)`
- `plan:internal/slackengine#EnsureRelayWhileAway` -> `func EnsureRelayWhileAway(geom Geometry, primeName string) (StartOutcome, error)`
- `plan:internal/slackengine#RelayLive` -> `func RelayLive(geom Geometry) (bool, error)`
- `plan:internal/slackengine#StopRelay` -> `func StopRelay(geom Geometry, callerPID int) ([]runproc.Ended, error)`
- `plan:internal/slackengine#SpawnRelayFromIntermediate` -> `func SpawnRelayFromIntermediate(geom Geometry, primeName string) error`
- `plan:internal/slackengine#AwaitReparent` -> `func AwaitReparent(intermediate int, parentPID func() int, clock Clock, bound time.Duration) error`
- `plan:internal/slackengine#AwaitFirstRead` -> `func AwaitFirstRead(geom Geometry, clock Clock, bound time.Duration) error`
- `plan:internal/slackengine#ErrSpawnUnderTest` -> `var ErrSpawnUnderTest = errors.New("slackengine: no relay spawned under test")`
- `plan:internal/slackengine#starter` -> `type starter struct`
- `plan:internal/slackengine#starter.ensure` -> `func (s starter) ensure(geom Geometry, primeName string, whileAway bool) (StartOutcome, error)`
- `plan:internal/slackengine#starter.spawnRelay` -> `func (s starter) spawnRelay(geom Geometry, primeName string) error`
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

`RunRelay` (in `relay.go`), told `RelayDeps{Mailbox, Slack, Hub, Clock, PID func() int, ReexecLockWait time.Duration, CheckBinary func(beforeExec func(), failed func(error)), ReapInherited func() error, Register func() error}`, `CheckBinary` in dobby's shape.
Production wiring (card 10) fills every field: `PID` is `os.Getpid`, `ReexecLockWait` 10 seconds, `CheckBinary` `selfexec.New().CheckWith`; a test sets every field, and a zero `ReexecLockWait` waits only `lock.ProbeGrace`.
1. Refuses through `RequireConfigured` with no token or setup file, naming `lym slack login` or `lym slack setup` (D3).
2. Calls `ReapInherited` (`proc.ReapInheritedChildren`) before it starts anything, then takes `LockPath`: a relay whose `RelayState.PID` equals `deps.PID()` is a re-exec and waits up to `ReexecLockWait` for the lock its old image released at the exec; any other start waits `lock.ProbeGrace`, as dobby's fresh start does, so a liveness probe (`RelayLive`, which takes the lock briefly) never makes a starting relay exit. A relay that still cannot take the lock exits with no error before its first poll, so a second start for the hub starts nothing and two relays never both poll.
3. Calls `Register` (`runproc.Register` under `runproc.HubOwner` with `runproc.SelfRecord(runproc.RoleSlackRelay)`) and writes only `RelayState`'s `PID`, `StartTime`, `FirstReadAt` and `FirstReadRefusal`: its pid and start time, `FirstReadAt` zero and no refusal, keeping `LastSlackOK` and the starter's `StartFailures`, `NextStartAt` and `LastSpawnAt` as they stand.
   Every `RelayState` write, the relay's and the starter's, is a read-modify-write under the record's lock through `internal/state`, so neither wipes the other's fields.
4. Makes its first creel read as `<shortname>:SLACK` before any Slack call: `Mailbox.ReadAll` with a handler that returns the package sentinel `errFirstReadAccepted`, so any mail stays unread for `outbound.read`; a read with no mail calls no handler and returns nil. Either nil or an error matching `errFirstReadAccepted` is an accepted read and writes `FirstReadAt`, which `AwaitFirstRead` reads; it leaves `StartFailures` as it stands, since the starter judges a relay by how long it lived, not by its first read.
   A refusal matching `creelengine.ErrProgramClaimInStrandPane`, this run's or the first after a re-exec, writes `FirstReadRefusal`, releases the lock and returns `ErrRelayInStrandPane` wrapping creel's message, with the way forward "start the relay with `lym slack start` from the operator's terminal or an orch's pane", so on Linux a relay run by hand from a non-orch strand's pane polls Slack for nothing and leaves the lock free.
   Off Linux creel resolves a program claim from the environment alone and never refuses it by pane, so this bound holds on Linux only (card 10).
   Any other refusal or failure, such as an unknown live pane process or a daemon that is down, writes `FirstReadRefusal`, is retried with the poll's backoff while the relay holds its lock and calls no Slack method, and the first accepted read clears the refusal.
5. Runs `outbound.read` in a goroutine and `inbound.poll` in the loop, each waiting its own backoff on the told clock: a 5-second floor, doubled after each empty pass up to 60 seconds, back to the floor on activity (a forwarded or answered message, a successful outbound post, or a presence change the poll reads). A `*RateLimitError` from either waits its `RetryAfter` and doubles the shared floor, up to 60 seconds, for the rest of the run (D4).
   Each successful Slack call writes `LastSlackOK`.
6. At each poll calls `CheckBinary` with a `beforeExec` that releases the lock, so a replaced binary re-execs the relay, which runs from step 2 again (D3), and a `failed` that takes the exec error, logs it, and then retakes the lock with `lock.AcquireWriteLockWithin(geom.LockPath, lock.ProbeGrace)`, so a `RelayLive` probe holding it for a moment does not end the relay; only when that wait fails, as when a relay a start launched in the window holds the lock, the old image logs it and exits, so only one relay polls.
7. Appends every refused or failed read and every Slack failure to `LogPath`, Warn once per episode in the process log, as `internal/logger`'s Level policy asks.

The start (D3), in `start.go`:
- `RelayLive` probes the lock: held means live.
- `EnsureRelay` and `EnsureRelayWhileAway` call `starter.ensure` over the production `starter`, whose `spawn` field runs `<LymExecutable> slack relay --hub-path <hub> --prime <primeName> --fork` from the hub directory, detached with `proc.Detach` in a new session, logs the spawn at Info and waits on that intermediate with `cmd.Wait`; under `go test` it returns `ErrSpawnUnderTest`. Tests build a `starter` with a fake `spawn` and `clock` and call `ensure` and `spawnRelay` on it, so every test keeps `t.Parallel`.
  The prime's worktree name is told by the caller, which knows its prime, and rides every relay argv as `--prime`, since `fabricengine.PrimeName` lists worktrees from a worktree's anchor and a hub path names none (card 10).
- `starter.ensure` answers `not-configured` with no error when the token or the setup file is missing, `live` when the relay is live, and otherwise starts it and answers `started`. With `whileAway`, it answers `terminal` and starts nothing unless `ReadPresence` reads `slack`, for the prime dobby's tick (card 12).
- The tick's restarts are capped (`PATTERN-child-processes`, a retry loop around a spawn caps its count) through `RelayState`'s `StartFailures`, `NextStartAt` and `LastSpawnAt`, which the relay's step 3 keeps, and judge each spawned relay by how long it lived against one named constant, `relayHealthyLifetime` (10 minutes), in `start.go`.
  Before it calls `spawn`, `starter.ensure` writes `LastSpawnAt`, the spawn time, and `NextStartAt` in one `RelayState` read-modify-write, and returns the spawn's error after that write, so a spawn that fails to start, as when `cmd.Start` fails, leaves a `LastSpawnAt` with no `LastSlackOK` after it, and a fast relay's `LastSlackOK` never lands before its own `LastSpawnAt`.
  A `whileAway` start that finds the relay down before `NextStartAt` spawns nothing, counts nothing and answers `waiting`, which card 12 handles as it does every outcome but `gave-up`.
  Once due, when a `LastSpawnAt` stands, it judges the last spawn by that relay's lifetime, `LastSlackOK` minus `LastSpawnAt`, where a zero `LastSlackOK` or one not after `LastSpawnAt` reads as zero: a relay whose Slack calls succeed writes `LastSlackOK` at each poll, at least once a minute outside a rate limit's wait.
  A lifetime under `relayHealthyLifetime` counts one failure, whether that relay never started, died before step 3 or died after its first read; a lifetime at or over it sets `StartFailures` to 0 and counts none.
  Unless the count reached five, it then writes `LastSpawnAt` and sets `NextStartAt` to the spawn time plus one minute, doubled for each counted failure, so 1, 2, 4, 8 or 16 minutes before the fifth failure ends in `gave-up`, and only then spawns; every spawn sets it, so the one-minute floor covers a starting relay's 30-second reparent wait and its first read, and a relay still starting is never judged.
  Each due start spawns once and judges only the spawn before it, so a spawn counts at most one failure.
  At the fifth failure in a row it spawns no more, answers `gave-up`, and posts nothing itself: its caller logs it at Warn once.
  Only relays that each lived under `relayHealthyLifetime`, in a row, end in `gave-up`; a re-exec writes no `LastSpawnAt`, so a relay that lived long across its re-execs is judged long-lived whatever its last image did.
  The count caps the loop of quick failures; a relay that lived `relayHealthyLifetime` ran as a relay, so its restart opens a new count, and restarts of relays that each live that long are bounded in rate, at most one per `relayHealthyLifetime`, not in number, for as long as presence stays `slack`.
  `EnsureRelay` without `whileAway`, the operator's or the bring-up's start, ignores the cap and resets it before it spawns.
- `SpawnRelayFromIntermediate` is the intermediate's step, `starter.spawnRelay` over the production spawn: it starts `<LymExecutable> slack relay --hub-path <hub> --prime <primeName> --after <its own pid>` detached, stdout and stderr on `LogPath`, logs the spawn and exits without waiting on it, the one release `PATTERN-child-processes` admits (card 1), so the relay is reparented out of the starter's tree, and out of any pane's tree, whoever starts it.
- `AwaitReparent` is the relay's wait before step 4 when started with `--after`: it reads its parent once a second until it is no longer the intermediate, and errors after the bound (30 seconds) naming the intermediate; the relay verb then exits with that error before taking the lock (card 10).
- `StopRelay` ends the relay through `runproc.EndRole(geom.RunProcDir, runproc.HubOwner, runproc.RoleSlackRelay, callerPID)` (`PATTERN-child-processes`).
- `AwaitFirstRead` reads `RelayState` once a second until the relay it names is live (its pid and start time match a running process) and has written `FirstReadAt`, and refuses after the bound (30 seconds) naming a live relay's recorded `FirstReadRefusal`, or `LogPath` when none is recorded. A state whose relay is gone is read as not yet written, whatever refusal it records: the double-forked relay is in no pane, so a gone relay's recorded refusal is never the strand-pane refusal of a relay this start launched, and refusing at once on it would name an earlier relay's refusal. Each refusal ends with the way forward "fix what it names, then run `lym slack away` again; the relay keeps its lock and retries" (D6).

`cmd/lym`'s `unboundedWaitFiles` adds `internal/slackengine/relay.go`; `pattern/PATTERN-daemon-wakeups.md`'s "What counts" list adds the relay's loops, from five seconds doubling to sixty; `pattern/PATTERN-child-processes.md`'s run-process-owner bullet names the Slack relay beside the detached board sync as a `runproc.HubOwner` process.

`internal/slackengine/relay_test.go` is tier0, with the in-memory fakes of `inbound_test.go`, a temp `Geometry`, a fake clock and distinct fake pids per run:
`TestRunRelay` rows — with no token, and with no setup file, it refuses naming `lym slack login` and `lym slack setup` and takes no lock; a first read accepted calls `Register` once, writes `FirstReadAt`, and only then does the fake Slack see a call; a first read with unread SLACK mail is accepted, polls, and leaves the mail unread for the outbound read; a first read refused with `ErrProgramClaimInStrandPane` returns `ErrRelayInStrandPane` naming `lym slack start`, makes no Slack call, and the lock is free for a later run of another pid, which takes it; a first read refused with creel's unknown-pane refusal keeps the lock, makes no Slack call, records the refusal, and after the fake mailbox accepts makes its first read and polls; a second `RunRelay` of another pid while the first holds the lock returns before any read; a run whose `RelayState.PID` equals its own pid takes a lock its old image releases within `ReexecLockWait`, while a fresh run against the same held lock returns; a lock held only for a probe's grace is taken by a fresh run; a `CheckBinary` that reports a replaced binary releases the lock in `beforeExec`, and one whose exec fails logs the exec error and retakes it in `failed`, a lock held only for a probe's grace included, or exits when another holder keeps it; a successful Slack call writes `LastSlackOK`; a first read accepted with `StartFailures` standing at 4 writes `FirstReadAt` and keeps `StartFailures` 4, `NextStartAt` and `LastSpawnAt`; the inbound wait doubles after empty polls up to 60 seconds, returns to 5 on activity, and a `429` waits its `Retry-After` and doubles the floor to 10.
`TestEnsureRelay` rows, over a `starter` with a fake spawn and a fake clock that each fake relay moves forward before it writes `FirstReadAt` or `LastSlackOK`, so no row rests on a time equal to `LastSpawnAt` — a hub with no token or no setup answers `not-configured` and spawns nothing; a live relay answers `live` and spawns nothing; a down relay answers `started` with one spawn of `slack relay --hub-path <hub> --prime <name> --fork`, and the fake spawn, reading `RelayState` when called, finds `LastSpawnAt` and `NextStartAt` already stored; `spawnRelay` spawns `slack relay --hub-path <hub> --prime <name> --after <pid>` once and does not wait on it; `whileAway` spawns for a down relay only while presence is `slack`, and with presence `terminal` spawns nothing, the state `lym slack stop` leaves (D3's criterion on the next dobby tick); a `whileAway` start after a relay that died with no Slack call waits a minute before the next spawn; `whileAway` starts a second apart inside that wait, the first minute after a spawn included, answer `waiting`, spawn nothing and count no failure; a fake spawn that writes a `RelayState` as a relay reaching step 3 would, keeping the starter's fields, and then dies before its first read, spawned by five `whileAway` starts in a row with the clock moved past each wait, makes the sixth start answer `gave-up`, and nothing spawns until a start without `whileAway` resets the cap; a fake relay that writes `FirstReadAt` and a `LastSlackOK` one minute after `LastSpawnAt` and then dies, spawned by five due `whileAway` starts, makes the sixth start answer `gave-up`; a relay that dies before any Slack call, with an earlier relay's `LastSlackOK` standing from before `LastSpawnAt`, still counts one failure; a fake spawn that returns an error, over six `whileAway` starts with the clock moved past each wait, is called exactly five times, each of those starts returning the spawn's error with `LastSpawnAt` written and `NextStartAt` rising between them, and the sixth answers `gave-up` without calling it; a relay with four counted failures whose next spawn's fake relay writes `LastSlackOK` eleven minutes after `LastSpawnAt` and then goes down makes the next due `whileAway` start spawn, with `StartFailures` 0 after it, and one whose `LastSlackOK` lands exactly ten minutes after `LastSpawnAt` counts no failure either.
`TestRelayWaits` rows — `AwaitReparent` returns once the parent changes and errors at the bound; `AwaitFirstRead` returns on a live relay's `FirstReadAt`; a stale state of a gone relay that recorded a refusal before the await began is ignored, and the await returns once a fresh live relay's `FirstReadAt` is written; it refuses at the bound naming a live relay's recorded refusal, or the log when none is recorded, and each refusal names `lym slack away`.

**ImpactSummary:** New process code inside the new package, plus the one declared wait-file list entry and two background-file lists naming the relay; no other caller until cards 10 to 13.
