# Card 8 — adopt-caller-follow

**Create:**
- `plan:internal/runproc#RoleAdopt` -> `const RoleAdopt = "adopt"`
- `plan:internal/adoptcli#callerDeps` -> `type callerDeps struct`
- `plan:internal/adoptcli#hubCallerDeps` -> `func hubCallerDeps(prime *lymcwd.Location) (callerDeps, error)`
- `plan:internal/adoptcli#runAdopt` -> `func runAdopt(deps callerDeps, req request, discardFlag bool, out io.Writer) int`
- `plan:internal/adoptcli#TestRunAdopt` -> `func TestRunAdopt(t *testing.T)`
- `plan:internal/adoptcli#TestSpawnersStartNothingUnderTest` -> `func TestSpawnersStartNothingUnderTest(t *testing.T)`
- `plan:internal/adoptcli#followWakeFloor` -> `const followWakeFloor = time.Second`
- `plan:internal/adoptcli#followWakeCeiling` -> `const followWakeCeiling = 8 * time.Second`
- `plan:internal/adoptcli#followDeps` -> `type followDeps struct`
- `plan:internal/adoptcli#follow` -> `func follow(deps followDeps, call adoptrecord.Request, workerPID int, since time.Time) (adoptrecord.Record, error)`
- `plan:internal/adoptcli#followCap` -> `func followCap(keepWarm func(strand string) time.Duration, strand string) time.Duration`
- `plan:internal/adoptcli#runsInOrch` -> `func runsInOrch(strand string) bool`
- `plan:internal/adoptcli#workerArgs` -> `func workerArgs(req adoptrecord.Request) []string`
- `plan:internal/adoptcli#workerCommand` -> `func workerCommand(executable, dir string, args []string) *exec.Cmd`
- `plan:internal/adoptcli#spawnWorker` -> `func spawnWorker(prime *lymcwd.Location, args []string) (pid int, exited <-chan struct{}, err error)`
- `plan:internal/adoptcli#discard` -> `func discard(hubPath string) (removed bool, err error)`
- `plan:internal/adoptcli#TestFollow` -> `func TestFollow(t *testing.T)`
- `plan:internal/adoptcli#TestDiscard` -> `func TestDiscard(t *testing.T)`
- `plan:internal/adoptcli#TestRunsInOrch` -> `func TestRunsInOrch(t *testing.T)`
- `plan:internal/adoptcli#TestWorkerArgs` -> `func TestWorkerArgs(t *testing.T)`

**Delete:**
- `internal/adoptcli#TestStartSuiteStartsNothingUnderTest`

**Edit:**
- `internal/adoptcli#Command`

**Extends:**
- `internal/runproc/doc.go`
- `cmd/lym#unboundedWaitFiles`
- `pattern/PATTERN-daemon-wakeups.md`

**Uses:**
- `plan:internal/adoptcli#validateRequest`
- `plan:internal/adoptcli#judgeStanding`
- `plan:internal/adoptcli#workerCmd`
- `plan:internal/adoptrecord#Read`
- `plan:internal/adoptrecord#Remove`
- `plan:internal/adoptrecord#ErrDecode`
- `plan:internal/adoptrecord#LockPath`
- `plan:internal/adoptrecord#Record.ReadState`
- `plan:internal/adoptrecord#Request.Command`
- `plan:internal/adoptrecord#WorkerAlive`
- `internal/fabricengine#RunProcDir`

**Intent:**
D1's caller, `--no-orch` and `--discard`.
`lym adopt` runs no step itself: it validates, starts a detached worker, and follows the record.

`Command` gains `--no-orch` and `--discard`, and its `Long` text gives D3's step order, the worker, the follow and the record.
Its `RunE` resolves the prime, builds `hubCallerDeps` and returns `runAdopt`'s exit code, so the tests drive every branch below through `runAdopt` over fake seams.
`callerDeps` holds the hub path, the caller's strand name, the step deps `validateRequest` reads, the record read, the liveness read, the clock, the sleep, `KeepWarm`, the discard and `Spawn`.
`Spawn` is `spawnWorker` in production; a fake records its args and returns a pid and an `exited` channel.
`runAdopt`, in order:
1. `--discard` (with no subject or other flag, else refused naming `lym adopt --discard` alone) runs `discard` and prints whether it removed a record.
2. Otherwise it builds the request.
   `NoOrch` is also set when `runsInOrch` reads this process's strand name (`agentname.StrandNameEnv`) as an orch: `agentname.Parse` with a role matching `agentname.RoleOrch`, numbered roles included.
   It runs `validateRequest`, refusing before anything spawns.
   It reads the record and judges it with `judgeStanding`:
   - follow spawns nothing and follows the record's worker, passing the record's pid as `workerPID` and a nil `exited`, which never closes, so the follow ends on the record's own state, `interrupted` covering a worker that dies;
   - other subjects are refused, naming the recorded subjects, the step, the re-run and `lym adopt --discard`;
   - fresh and resume start a worker through `spawnWorker` with `workerArgs`.
3. It follows through `follow` and prints the end: the record's envelope through `output.Ok` for `done`, exit 0;
   `output.ErrFields` for `stopped`, naming the stop's way forward, exit non-zero;
   for `interrupted`, an error naming the re-run (`Request.Command`) and `lym adopt --discard`.

`spawnWorker` refuses under a test binary, as `startSuite` does.
It resolves the binary through `selfexec.LymBinary` and builds `workerCommand`, run in the prime, detached into its own session through `proc.Detach`.
It starts it and logs the spawn through `logger.Info`.
It waits on it in a goroutine that closes `exited` when it ends, and registers it under `runproc.HubOwner` in `fabricengine.RunProcDir` with `runproc.ChildRecord(pid, runproc.RoleAdopt, args)`.
A failed registration is logged and changes nothing.
`spawnWorker` refuses under a test binary, so the registration and the spawn log are held by review, as `PATTERN-child-processes`' Enforcement says ("Review covers the rest: that each spawn registers").
That the worker outlives its caller rests on `proc.Detach`'s new session, which `TestDetachSetsSetsid` pins.
`runproc.RoleAdopt` is the new role, and the runproc `doc.go` names it among the hub owner's roles.
The worker's stdout and stderr go to `adopt.out` beside the record, so a losing worker's envelope and a panic stay on disk.

`follow` loops, each wake reading the record:
- A step that ended since the last read prints one line on the command's stderr (`adopt: <step> done`).
- A record of `workerPID` that reads `done` or `stopped` returns it; one that reads `interrupted` returns the error above.
- Once `exited` is closed and the record names another pid:
  that worker lost the lock, and `follow` takes the other record when its start is later than `since`, the caller's own start, and its subjects match.
  Other subjects are refused as D2 refuses; a record not started later is not followed.
- Once `exited` is closed with no such record, it reads once more at the next wake.
  With still none it returns an error naming the re-run of the same `lym adopt`, `lym beam status` and `adopt.out`, where a worker that refused or lost the lock left its envelope; a `--discard` that removed the record ends here.
- The wake starts at `followWakeFloor`, doubles to `followWakeCeiling` while the record is unchanged, and returns to the floor on a change (`PATTERN-daemon-wakeups`).
- At the cap it returns `still running`, naming `lym beam status` and the re-run of the same `lym adopt` to follow again; the caller exits non-zero.
`followCap` returns zero, uncapped, for an empty strand, a plain shell, where the follow waits until the worker ends, without calling `keepWarm`; otherwise the cap `keepWarm` answers.
`hubCallerDeps` fills `KeepWarm` with the `Cap` of `shuttleengines.KeepWarm` over the prime's config dirs.
`followDeps` holds the record read, the liveness read, `exited`, the clock, the sleep, the cap and the writer the step lines go to, so tests never wait out an interval and see each line.

`discard` takes `adopt.lock` non-blocking.
While another process holds it, it refuses naming `lym beam status`, and the record stands.
Holding the lock, it removes an unfinished record, and a record whose read `errors.Is` `adoptrecord.ErrDecode`, so the way forward an undecodable record names progresses; a done record, or none, is left and reported as nothing removed.
`--discard` lifts no pause and undoes no owner write.

What the change lets through: `--no-orch` skips only the orch start, bounded as card 2 says.
`--discard` drops a record's resume, bounded by the lock: it never removes a running worker's record.
A shell-run caller with no cap follows until the worker ends; the worker outlives any caller.

Forced registrations: the follow joins `unboundedWaitFiles` in `cmd/lym/daemonwakeups_test.go`, since it waits with no bound in a plain shell.
The follow is also listed among the unbounded waits of `pattern/PATTERN-daemon-wakeups.md`'s "What counts", from one second doubling to eight while the record is unchanged, capped by the keep-warm cap in an agent session.

Tests, tier0:
- `TestFollow` over fake records, liveness and clock, one row per D1 criterion:
  - steps print as they end, and done returns exit-ok data;
  - stopped and interrupted return their errors, interrupted naming the re-run and `lym adopt --discard`;
  - a worker that lost the lock to a later-started resuming worker of the same subjects is followed to its end;
  - one that lost it to a later worker of other subjects is refused naming the running adopt's subjects;
  - a record of another pid started before `since` is not followed, and the follow ends as for no record;
  - an exited worker with no record reads once more, then errors naming the re-run, `lym beam status` and `adopt.out`, as after a `--discard`;
  - at the cap it returns `still running` naming `lym beam status`, and with cap zero it waits on;
  - an unchanged record doubles the wake to the ceiling, and a change returns it to the floor.
- `TestDiscard` over `t.TempDir()`: removes an interrupted record; removes an undecodable one; refuses while another lock holder holds `adopt.lock`, naming `lym beam status` and leaving the record; leaves a done record; reports nothing removed with no record.
- `TestRunAdopt`, over fake `callerDeps` and a record in `t.TempDir()`, one row per verb-level branch:
  - `--discard` with a subject is refused naming `lym adopt --discard`, spawning nothing;
  - a live record of the same subjects spawns nothing and follows it, exiting 0 when it ends `done`;
  - a record that ends `stopped` exits non-zero with its way forward;
  - one that reads `interrupted` exits non-zero naming the re-run and `lym adopt --discard`;
  - an unfinished record of other subjects is refused, spawning nothing;
  - from an orch strand the spawned args carry `--no-orch`, and from an implementer strand they do not;
  - an empty strand follows uncapped without calling `KeepWarm`, and an agent strand at `KeepWarm`'s cap exits non-zero with `still running`.
- `TestRunsInOrch`: an orch strand and a numbered orch read true; an implementer strand, an empty name and an unparsable name read false.
- `TestWorkerArgs`: each request renders to args that `workerCmd`'s flags parse back to the same request.
- `TestSpawnersStartNothingUnderTest` replaces `TestStartSuiteStartsNothingUnderTest` with one row per spawner, `startSuite` and `spawnWorker`: each returns an error and starts nothing under the test binary.

**ImpactSummary:** the `lym adopt` verb's entry changes from an in-process run to spawn-and-follow inside adoptcli; runproc gains one role constant, and the wakeups guard list and background gain one line each.
