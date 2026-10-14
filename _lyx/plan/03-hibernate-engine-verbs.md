# Card 3 — hibernate-engine-verbs

**Create:**
- `internal/hibernateengine/doc.go`
- `internal/hibernateengine/request.go`
- `internal/hibernateengine/verbs.go`
- `internal/hibernateengine/testmain_test.go`
- `internal/hibernateengine/world_test.go`
- `internal/hibernateengine/verbs_test.go`
- `plan:internal/hibernateengine#Trigger` -> `type Trigger string`
- `plan:internal/hibernateengine#TriggerNow` -> `const TriggerNow Trigger = "now"`
- `plan:internal/hibernateengine#TriggerAfter` -> `const TriggerAfter Trigger = "after"`
- `plan:internal/hibernateengine#TriggerAfterAll` -> `const TriggerAfterAll Trigger = "after-all"`
- `plan:internal/hibernateengine#Phase` -> `type Phase string`
- `plan:internal/hibernateengine#PhaseWaiting` -> `const PhaseWaiting Phase = "waiting"`
- `plan:internal/hibernateengine#PhasePausing` -> `const PhasePausing Phase = "pausing"`
- `plan:internal/hibernateengine#PhaseConfirming` -> `const PhaseConfirming Phase = "confirming"`
- `plan:internal/hibernateengine#PhaseHibernating` -> `const PhaseHibernating Phase = "hibernating"`
- `plan:internal/hibernateengine#PhaseResuming` -> `const PhaseResuming Phase = "resuming"`
- `plan:internal/hibernateengine#PhaseDone` -> `const PhaseDone Phase = "done"`
- `plan:internal/hibernateengine#PhaseFailed` -> `const PhaseFailed Phase = "failed"`
- `plan:internal/hibernateengine#PhaseCancelled` -> `const PhaseCancelled Phase = "cancelled"`
- `plan:internal/hibernateengine#PauseCaller` -> `const PauseCaller = "hibernate"`
- `plan:internal/hibernateengine#Request` -> `type Request struct`
- `plan:internal/hibernateengine#Request.Terminal` -> `func (r Request) Terminal() bool`
- `plan:internal/hibernateengine#RunStatus` -> `type RunStatus struct`
- `plan:internal/hibernateengine#Deps` -> `type Deps struct`
- `plan:internal/hibernateengine#errUnchanged` -> `var errUnchanged = errors.New("hibernateengine: the request is unchanged")`
- `plan:internal/hibernateengine#readRequest` -> `func readRequest(path string) (Request, bool, error)`
- `plan:internal/hibernateengine#updateRequest` -> `func updateRequest(path string, mutate func(cur Request, found bool) (Request, error)) error`
- `plan:internal/hibernateengine#daemonLive` -> `func daemonLive(deps Deps, r Request) bool`
- `plan:internal/hibernateengine#refuseCaller` -> `func refuseCaller(strand string) error`
- `plan:internal/hibernateengine#refusePlatform` -> `func refusePlatform(goos string) error`
- `plan:internal/hibernateengine#liftOwned` -> `func liftOwned(deps Deps, r Request) (*tenderengine.ResumeReport, error)`
- `plan:internal/hibernateengine#SubmitRequest` -> `type SubmitRequest struct`
- `plan:internal/hibernateengine#Submit` -> `func Submit(deps Deps, req SubmitRequest) (Request, error)`
- `plan:internal/hibernateengine#CancelReport` -> `type CancelReport struct`
- `plan:internal/hibernateengine#Cancel` -> `func Cancel(deps Deps) (CancelReport, error)`
- `plan:internal/hibernateengine#StatusReport` -> `type StatusReport struct`
- `plan:internal/hibernateengine#Status` -> `func Status(deps Deps) (StatusReport, error)`
- `plan:internal/hibernateengine#fakeWorld` -> `type fakeWorld struct`
- `plan:internal/hibernateengine#newFakeWorld` -> `func newFakeWorld(t *testing.T) *fakeWorld`
- `plan:internal/hibernateengine#fakeWorld.deps` -> `func (w *fakeWorld) deps() Deps`
- `plan:internal/hibernateengine#TestSubmit` -> `func TestSubmit(t *testing.T)`
- `plan:internal/hibernateengine#TestCancel` -> `func TestCancel(t *testing.T)`
- `plan:internal/hibernateengine#TestStatus` -> `func TestStatus(t *testing.T)`

**Edit:**
- `cmd/lyx/testdata/test-budget.yaml`

**Uses:**
- `plan:internal/tenderengine#Lift`
- `plan:internal/tenderengine#EntryRef`
- `internal/tenderengine#PauseRequest`
- `internal/tendercli#callerOf`

**Intent:**
Create the hibernate engine's request file and its three operator verbs: submit, cancel and status.
The daemon sequence (`Watch`) lands in card 4 over the same `Deps`.
Every path and probe is told through `Deps` (`PATTERN-told-geometry`): the engine resolves no working directory, spawns nothing itself and imports no `lyxcwd`.

**request.go.**
`Request` is the hub's one hibernate request, one JSON file at `Deps.RequestPath` with its lock at `RequestPath + ".lock"`.
Its fields and JSON keys: `Trigger` (`trigger`), `Slug` (`slug`, the `--after` slug), `Requester` (`requester`, the caller's strand name or `EXTERNAL`), `RequestedAt` (`requested_at`), `DaemonPID` (`daemon_pid`), `DaemonStart` (`daemon_start`), `Owned` (`owned`, a `*tenderengine.EntryRef`, omitted when nil), `Phase` (`phase`) and `Reason` (`reason`, the last reason, empty when none).
`Terminal` reports `done`, `failed` or `cancelled`.
`updateRequest` is the one writer: it runs `state.UpdateJSON` over the file and its lock, so every write holds the request lock across its read and write and keeps keys a newer binary added (`PATTERN-tolerant-records`); a mutate returning `errUnchanged` writes nothing and `updateRequest` returns nil.
`readRequest` decodes the file with a plain read and `json.Unmarshal`, read-only and with no lock, a missing file reading as not found.
`daemonLive` reports a non-zero `DaemonPID` for which `Deps.Alive(pid, start)` holds, so a recycled pid is not taken for the daemon.
`RunStatus` is one batten run's status as the engine reads it: `State` (`shedengine.State`) and `Producer` (the current row's name).

`Deps` holds, all told: `RequestPath`; `Caller`, the running process's strand name, empty for an external caller; `GOOS`; `LookPath func(file string) (string, error)`; `Spawn func() (pid int, startTime string, err error)`, which starts the detached daemon; `SelfPID int` and `SelfStart string`, the running process's own identity; `Alive func(pid int, startTime string) bool`; `Kill func(pid int) error`; `RunStatus func(slug string) (RunStatus, bool, error)` and `RunSlugs func() ([]string, error)` over the prime's batten runs; `LandLockHeld func() (bool, error)`; `Pause func(tenderengine.PauseRequest) (tenderengine.PauseReport, error)`, `TenderStatus func() (tenderengine.StatusReport, error)` and `Lift func(tenderengine.EntryRef) (tenderengine.ResumeReport, error)` over tender's engine; `Hibernate func() error`; `Notify func(line string) error`, which queues one orch notice; `Now`, `Sleep` and `Awake func() time.Duration`, the awake clock.
`PauseCaller` is the caller label the daemon's pause entry carries, which card 5 hands `tendercli.HubDeps` as the strand.

**Callers and platform.**
`refuseCaller` follows tendercli's rule (`callerOf`): an empty strand is external, a strand `agentname.Parse` reads as role `orch` with no slug is the orch, and any other strand, a name that does not parse included, is an agent, refused with `hibernate: an agent may not request or cancel a hibernate; way forward: ask your parent session, which runs "lyx hibernate request" or "lyx hibernate cancel" for you`.
Bound: it guards against a mistaken call, as tender's check does; the agents' settings deny any command naming the strand variable, so an agent cannot clear it to read as external.
`refusePlatform` refuses any `GOOS` but `linux` with `hibernate: lyx hibernates the machine on Linux only, and this machine runs <goos>; way forward: hibernate it from the desktop yourself`.

**Submit.**
`Submit(deps, SubmitRequest{After string, AfterAll bool})` checks in this order, each refusal writing nothing and spawning nothing: the caller; the platform; `LookPath("systemctl")`, refused with `hibernate: systemctl is not on PATH, so lyx cannot hibernate this machine; way forward: put systemctl on PATH, then re-run "lyx hibernate request"`; `After` beside `AfterAll`, refused with `hibernate: --after and --after-all are exclusive; way forward: pass one of them, or neither to hibernate once the pause is confirmed`; and for `After`, a slug `RunStatus` does not find, refused with `hibernate: %q has no batten status in this hub; way forward: name one of the slugs that has one: <RunSlugs joined by ", ">`, or `no batten run has one yet` for none.
Then, in one `updateRequest` hold: a non-terminal request with a live daemon is refused with `hibernate: a hibernate request is pending in phase <phase> with daemon pid <pid>; way forward: run "lyx hibernate cancel", then request again`; a non-terminal request whose daemon is dead first has its owned entry lifted through `liftOwned` (a lift error aborts the hold and writes nothing); then `Spawn` runs, and the new request is written with the trigger (`now`, `after` with its slug, or `after-all`), the requester, `Now()`, the spawned pid and start time, no owned entry and phase `waiting`.
A terminal request, and no request, are replaced the same way.
Because the check, the spawn and the write share one hold, two racing submits spawn one daemon.
A spawn error aborts the hold, writes nothing and is returned.
`Submit` returns once the daemon is spawned; the daemon itself decides when the trigger holds.

**Cancel.**
`Cancel(deps)` refuses an agent caller and a non-Linux platform as `Submit` does.
In one `updateRequest` hold it reads the request: none, or a terminal one, writes nothing and reports nothing cancelled, which succeeds.
A live daemon in `hibernating` is refused with `hibernate: the machine is going into hibernation; way forward: none until it wakes, when the daemon lifts its own pause; run "lyx hibernate status" to follow it`.
Otherwise the request is written `cancelled` with the reason `cancelled by <requester label>`, keeping its other fields.
After the hold, a daemon that read live is stopped through `Kill(pid)`; a dead daemon's request is cancelled in any phase, `hibernating` included, which is the way out after a crash or a cold boot left the hub paused.
Then the request is read again, since the daemon may have recorded its owned entry after the hold and before the kill, and `liftOwned` lifts that entry through `Lift`, resume lines included; a request with no owned entry lifts nothing, so a hub pause that stood before the request stays.
Bound: a daemon killed between its pause's record write and its `Added` write (card 4) leaves its entry live and unowned, which `lyx resume` lifts; the window is one file write.
`CancelReport` carries `Cancelled` (`cancelled`), the phase it was cancelled from (`phase`), `DaemonStopped` (`daemon_stopped`) and the lift's `*tenderengine.ResumeReport` (`lifted`, omitted when nothing was lifted).

**Status.**
`Status(deps)` takes no caller and no platform check and writes nothing: it reads the request through `readRequest` and reports `StatusReport{Request *Request "request", DaemonLive bool "daemon_live"}`, with `request` null when there is none.
The request carries its trigger, slug, phase, reason and owned entry, so status reports a halted `--after` run's reason (card 4) and a dead daemon as `daemon_live: false`.

**doc.go** states the package: the request file and its lock, the phases, the caller rule with its bound, the platform rule, and the submit, cancel and status rules above, each with its bound; card 4 adds the daemon's sequence.

**Tests** (tier 1, no spawn, no `systemctl`; `testmain_test.go` runs `tmuxkit.Main` as `tenderengine`'s does).
`world_test.go` declares `fakeWorld`, shared with card 4's `TestWatch`: a request path and a pause record path under `t.TempDir()`; a wall clock and an awake clock that `Sleep` both advance, with a pending-sleep switch that makes the next `Sleep` move the wall clock an hour past the awake clock; the caller strand, the GOOS and whether `systemctl` is found; a mutex guarding every field the seams touch, so parallel goroutines race on nothing; a spawn counter that hands out pids and records each spawned pid alive with a start time, with a configured spawn error; the self pid and start; an alive map, a kill list and an `onKill` hook a row uses to act as the daemon before it dies; batten runs keyed by slug, each a list of readings consumed per call and the last repeated, with a missing-slug switch; a land-lock script; a fake `Pause` that adds a hub entry with caller `PauseCaller` through `pauserecord.Update` unless one is live (then `AlreadyPaused`), calls `req.Added` with the added entry, and returns a configured `StillBusy`, `Refused` and `ResumedDuringGrace`, or a configured error; a fake `TenderStatus` whose `Live` is the pause record's live entries and whose `Sessions` and `NetOps` come from a per-reading script, with a configured reading error and an `onStatus` hook called with the call's index before it reads, so a row acts between two readings with no `Sleep` between them; `Lift` wired to the real `tenderengine.Lift` over `tenderengine.Deps{RecordPath, Now, Sleep}`, so a lift by identity acts on the record, unless a configured lift error replaces it; a list of every duration `Sleep` was given; a `Hibernate` call counter with a configured error and the pending-sleep switch; a notice list; and an `onSleep` hook a row uses to act mid-sequence.
`deps()` returns the `Deps` over them.

`TestSubmit` rows:
- no trigger spawns once and writes phase `waiting`, trigger `now`, the requester, the spawned pid and start time and no owned entry, and returns that request;
- `--after <slug>` with a found status writes trigger `after` and the slug, and `--after-all` writes trigger `after-all`;
- an agent caller (`lyx:p1:driver`) is refused naming the parent session, and the orch (`lyx:orch`) and an external caller are not;
- a non-Linux GOOS, a missing `systemctl`, `--after` beside `--after-all`, and a slug with no batten status listing the slugs that have one, are each refused with no spawn and no request file;
- a non-terminal request with a live daemon is refused naming `lyx hibernate cancel`, the file unchanged;
- a non-terminal request with a dead daemon and an owned entry live in the record has that entry lifted (the record's live list empties) and is replaced by a fresh `waiting` request with the new pid;
- each of a `done`, `failed` and `cancelled` request is replaced with one spawn;
- two submits run in parallel goroutines spawn exactly one daemon, and the other is refused naming `lyx hibernate cancel`;
- a spawn error is returned with no request file written;
- over a dead daemon's request, a lift error is returned and the old request is left byte-for-byte as it was, with no spawn.

`TestCancel` rows:
- with no request, and with a terminal one, it succeeds with `cancelled: false`, kills nothing and leaves the file as it was;
- a `waiting` request with a live daemon is written `cancelled`, its daemon killed, and nothing lifted, for the orch caller and for an external caller alike;
- a `confirming` request with a live daemon and an owned live entry is cancelled, the daemon killed, the entry lifted from the record and the lift reported;
- an owned entry the daemon writes into the file after the hold, written from the `onKill` hook, is still lifted;
- a `hibernating` request with a live daemon is refused naming `lyx hibernate status`, and nothing is killed or lifted;
- a `hibernating` request with a dead daemon is cancelled and its owned entry lifted;
- a request with no owned entry, while a hub entry another caller added is live, lifts nothing and that entry stays live;
- an agent caller and a non-Linux GOOS are refused.

`TestStatus` rows: no request reports `request` null; a request reports its trigger, slug, phase, reason and owned entry, with `daemon_live` true for a live pid and false for a dead one; the file's bytes and the directory listing are unchanged after the call; and a non-Linux GOOS still reads.

`cmd/lyx/testdata/test-budget.yaml` gains the row `internal/hibernateengine: 3`.

**ImpactSummary:** a new package with one budget row; nothing existing calls it yet.

**Commit:** 3: hibernate: the request file and the request, cancel and status sequences
