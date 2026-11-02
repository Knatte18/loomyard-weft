# Card 1 — gatefixer-core

**Create:**
- `internal/gatefixer/doc.go`
- `internal/gatefixer/testmain_test.go`
- `plan:internal/gatefixer#RoleGateFixer` -> `const RoleGateFixer = "gate-fixer"`
- `plan:internal/gatefixer#RoleVerifyFixer` -> `const RoleVerifyFixer = "verify-fixer"`
- `plan:internal/gatefixer#Roles` -> `func Roles() []string`
- `plan:internal/gatefixer#IsFixerRole` -> `func IsFixerRole(role string) bool`
- `plan:internal/gatefixer#StatusRunning` -> `const StatusRunning = "running"`
- `plan:internal/gatefixer#StatusDone` -> `const StatusDone = "done"`
- `plan:internal/gatefixer#StatusDead` -> `const StatusDead = "dead"`
- `plan:internal/gatefixer#DeadReasonAsking` -> `const DeadReasonAsking = "asking"`
- `plan:internal/gatefixer#DeadReasonTimeout` -> `const DeadReasonTimeout = "timeout"`
- `plan:internal/gatefixer#DeadReasonDied` -> `const DeadReasonDied = "died"`
- `plan:internal/gatefixer#Record` -> `type Record struct`
- `plan:internal/gatefixer#Store` -> `type Store interface`
- `plan:internal/gatefixer#Starter` -> `type Starter interface`
- `plan:internal/gatefixer#Stopper` -> `type Stopper interface`
- `plan:internal/gatefixer#Engines` -> `type Engines interface`
- `plan:internal/gatefixer#Clock` -> `type Clock interface`
- `plan:internal/gatefixer#Deps` -> `type Deps struct`
- `plan:internal/gatefixer#Spec` -> `type Spec struct`
- `plan:internal/gatefixer#Result` -> `type Result struct`
- `plan:internal/gatefixer#SpawnOrAttach` -> `func SpawnOrAttach(deps Deps, store Store, spec Spec) (*Record, bool, error)`
- `plan:internal/gatefixer#Await` -> `func Await(deps Deps, rec *Record, timeout, wait time.Duration) (*Result, error)`
- `plan:internal/gatefixer#StrandLive` -> `func StrandLive(reed shuttleengine.ReedOps, guid string) (bool, error)`
- `plan:internal/gatefixer#TurnEndRead` -> `type TurnEndRead struct`
- `plan:internal/gatefixer#TurnEndedAfter` -> `func TurnEndedAfter(eventsPath string, offset int64, engine shuttleengine.Engine, read TurnEndRead) (bool, error)`
- `plan:internal/gatefixer#Registrar` -> `func Registrar(reed shuttleengine.ReedOps, runProcDir, owner, role string) func(guid string) error`
- `plan:internal/gatefixer#TestIsFixerRole` -> `func TestIsFixerRole(t *testing.T)`
- `plan:internal/gatefixer#TestSpawnOrAttach` -> `func TestSpawnOrAttach(t *testing.T)`
- `plan:internal/gatefixer#TestAwait` -> `func TestAwait(t *testing.T)`
- `plan:internal/gatefixer#TestStrandLive` -> `func TestStrandLive(t *testing.T)`
- `plan:internal/gatefixer#TestTurnEndedAfter` -> `func TestTurnEndedAfter(t *testing.T)`
- `plan:internal/gatefixer#TestRegistrar` -> `func TestRegistrar(t *testing.T)`

**Edit:**
- `PATTERN.md`

**Uses:**
- `internal/websterengine#awaitColdFixer`
- `internal/websterengine#TestStrandLive`
- `internal/websterengine#TestTurnEnded`
- `internal/loomcli#sessionRegistrar`
- `internal/testkit/shuttlefake#Reed`
- `internal/testkit/shuttlefake#Engine`

**Intent:**
Create `internal/gatefixer`, the one owner of a cold fixer strand's life (D1): spawn-or-attach, the bounded wait, the stop and the runproc registration.
It is lifted from `internal/websterengine/coldfixer.go` and the spawn half of `internal/websterengine/verifyfixer.go`; card 3 moves webster onto it and deletes the originals.
`gatefixer` imports nothing of `internal/websterengine`, so `internal/gatecli` can import it (D4).

- Roles: `RoleGateFixer` (`gate-fixer`, the four new callers) and `RoleVerifyFixer` (`verify-fixer`, webster's, moved here by card 3) are the set of fixer roles; `Roles()` returns both in that order and `IsFixerRole(role)` reports whether `role` matches one of them through `agentname.MatchesRole`, so `gate-fixer-2` and `verify-fixer-3` match and `gate-fixer-1`, `implementer` and the empty role do not.
  `SpawnOrAttach` refuses a `Spec.Role` outside the set, naming the role and the way forward to spawn under `RoleGateFixer` or `RoleVerifyFixer`; that branch is a fail-loud wiring guard, since every caller passes a constant.
- `Record` holds `StrandGUID`, `SessionID`, `Engine`, `SpawnedAt` (RFC 3339), `ShuttleRunDir`, `EventsPath`, `EventsOffset`, `ReportPath`, `ReplyPath`, `Status` and `DeadReason`, with JSON tags `strandGuid`, `sessionId`, `engine`, `spawnedAt`, `shuttleRunDir`, `eventsPath`, `eventsOffset`, `reportPath`, `replyPath`, `status`, `deadReason`, every one but `strandGuid` and `spawnedAt` `omitempty`.
  A record is live while `Status` is empty.
- `Store` is the caller's store of records: `Live() (*Record, error)` returns the newest record with no terminal status, nil when none, and `Save(rec *Record) error` appends or rewrites the record of `rec.StrandGUID`.
- `Starter` (`Start(shuttleengine.Spec) (*shuttleengine.Run, error)`), `Stopper` (`StopStrand(guid string) error`), `Engines` (`Lookup(name string) (shuttleengine.Engine, error)` and `DefaultName() string`) and `Clock` (`Now() time.Time`, `Sleep(time.Duration)`) carry the method sets webster's `Starter`, `StrandStopper`, `AuditEngines` and `Clock` already have, so webster's values satisfy them unchanged.
- `Deps` carries `Starter`, `Stopper`, `Reed shuttleengine.ReedOps`, `Engines`, `ShuttleCfg shuttleengine.Config`, `AnchorRoot string`, `Clock` and `Register func(guid string) error`; a nil `Register` leaves the strand unregistered, which only webster's standalone run passes (D1).
- `Spec` carries `Prompt []byte`, `ReportPath`, `ReplyPath`, `Role`, `ModelChoice shuttleengine.ModelChoice`, `Skills []string`, `Segment segmentcolor.Segment` and `Timeout time.Duration`.
- `SpawnOrAttach(deps, store, spec)`: when `store.Live()` returns a record, it returns it with `false` and spawns nothing.
  Otherwise it removes a stale reply file at `spec.ReplyPath`, starts the strand through `deps.Starter` with the reply file as its only output file, resolves its run with `shuttleengine.FindRun(deps.ShuttleCfg, deps.AnchorRoot, guid)`, registers it through `deps.Register` when set, builds the record with `SpawnedAt` from `deps.Clock`, and saves it through `store.Save` before it returns it with `true`.
  A start failure is returned wrapped as transient; a run that cannot be resolved, a failed registration or a failed save stops the strand through `deps.Stopper` and returns the joined error, wrapped as transient, so no unrecorded or unregistered strand outlives the call.
  Every caller names its own way forward on these errors: webster's the transient re-run of its verb (card 3), the row wrap's a re-prompt of the writer (card 6) and the card gate's the call's red verdict (card 12).
- `Await(deps, rec, timeout, wait)` polls once a second on `deps.Clock` until a terminal classification or `wait` elapses, exactly as `awaitColdFixer` does: done once `rec.ReplyPath` exists, else dead `asking` when the strand's newest turn ended (`TurnEndedAfter` over `rec.EventsPath` from `rec.EventsOffset`, with the engine `rec.Engine` names through `deps.Engines`), dead `timeout` once `timeout` has passed since `rec.SpawnedAt`, dead `died` when `StrandLive` reports the strand gone, checked in that order.
  A wait that elapses returns `StatusRunning`.
  Every terminal result stops the strand through `deps.Stopper`; done also reads the reply into `Result.Reply` and removes `rec.ShuttleRunDir`; a failed stop, read or removal is a `Result.Warnings` line.
  `Result` carries `Status`, `DeadReason`, `Reply`, `ElapsedS` and `Warnings`.
  `Await` never marks the record terminal: the caller does, through its store, at the point D1 names for it.
- `StrandLive`, `TurnEndRead` and `TurnEndedAfter` are webster's same-named members with their bodies and doc comments unchanged but their error prefix `gatefixer:`; card 3 deletes webster's copies.
- `Registrar(reed, runProcDir, owner, role)` returns the `Deps.Register` seam, or nil for an empty `owner`, so a caller on the prime or outside a hub passes its empty pair slug and gets no registration: the seam reads reed's strand table, finds the strand by guid and registers its pane process with `runproc.Register(runProcDir, owner, runproc.ChildRecord(panePID, role, nil))`, so `runproc.End`, `EndSparing` and `EndRole` stop the fixer; a strand absent from the table or with no pane process is an error, as `sessionRegistrar` refuses it.
- `doc.go` documents the package: the fixer role set, the record and store, the spawn, wait and stop, and that a fixer never judges its gate.

`internal/gatefixer/testmain_test.go` is untagged: it holds only `TestMain`, running the package under `tmuxkit.Main`.

Tests, all at tier 0 (`tiergate.Require(t, 0)`), in package `gatefixer` over `shuttlefake.Reed`, `shuttlefake.Engine`, a recording starter and stopper, an in-memory store and a fake clock:
- `TestIsFixerRole`: rows for both base roles, numbered forms, `-1`, a leading-zero form, another role and the empty role.
- `TestSpawnOrAttach`: a live record attaches and starts nothing; no live record spawns with the reply file as output file, removes a stale reply, saves before returning and calls `Register` with the guid; a nil `Register` registers nothing; a start failure saves nothing and stops nothing; a failed `FindRun`, a failed `Register` and a failed `Save` each stop the strand; a role outside the set is refused.
- `TestAwait`: done on a reply file with the reply read and the run dir removed; dead asking, timeout and died in that order of precedence; running when the wait elapses; every terminal result stops the strand once; a failed stop is a warning; an engine `rec.Engine` names that `deps.Engines` does not hold and an unparsable `SpawnedAt` are each returned as errors, stopping nothing.
- `TestStrandLive` and `TestTurnEndedAfter`: webster's `TestStrandLive` and `TestTurnEnded` rows, moved here.
- `TestRegistrar`: a live strand registers its pane pid under the owner with the role, the row reading the record file under `<dir>/<owner>/` directly, since `runproc`'s own read prunes a record whose process is gone; an empty owner returns a nil seam; an absent strand and a strand with pane pid 0 are errors; a status error propagates.

`PATTERN.md` gains this entry in its `## Shed and loom` section, after `PATTERN-gate-slot`:

```
- `PATTERN-gate-fixer` — Clearing a failed gate with a cold agent: only `gatefixer`, one fixer per failed evaluation; Go's re-run of the gate judges, never the fixer; a fixer's own gate calls spawn none.
```

**ImpactSummary:** a new package with no caller yet, plus one PATTERN entry; nothing existing changes behavior.
