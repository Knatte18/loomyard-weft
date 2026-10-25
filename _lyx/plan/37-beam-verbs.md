# Card 37 — beam-verbs

**Edit:**
- `internal/beamcli#Command`
- `internal/beamcli#createDeps`
- `internal/beamcli#createResult`
- `internal/beamcli#createSteps`
- `internal/beamcli#promoteProject`
- `internal/beamcli#promoteSteps`
- `internal/beamcli#hubCreateDeps`
- `internal/beamcli#newCreateWorld`
- `internal/beamcli#createWorld.deps`
- `internal/beamcli#TestCreate`
- `internal/beamcli#newPromoteWorld`
- `internal/beamcli#TestPromote`
- `internal/beamcli/doc.go`
- `docs/overview.md`
- `PATTERN.md`

**Create:**
- `plan:internal/beamcli#dobbyProgram` -> `const dobbyProgram = "DOBBY"`
- `plan:internal/beamcli#startCmd` -> `func startCmd() *cobra.Command`
- `plan:internal/beamcli#dobbyCmd` -> `func dobbyCmd() *cobra.Command`
- `plan:internal/beamcli#hubDobbyDeps` -> `func hubDobbyDeps(prime *lyxcwd.Location, geom dobbyengine.Geometry) (dobbyengine.Deps, error)`
- `plan:internal/beamcli#registerDobby` -> `func registerDobby(runProcDir string, geom dobbyengine.Geometry) error`
- `plan:internal/beamcli#TestRegisterDobby` -> `func TestRegisterDobby(t *testing.T)`

**Uses:**
- `plan:internal/dobbyengine#Run`
- `plan:internal/dobbyengine#EnsureLive`
- `plan:internal/hubgeom#DobbyStart`
- `plan:internal/hubgeom#DobbyGeometry`
- `plan:internal/runproc#RoleDobby`
- `plan:internal/orchcli#NotifyFromScoped`

**Intent:**
Add `lyx beam start` and the hidden `lyx beam dobby`, and give every beam a dobby from birth (D1, D2).
`startCmd` (operator audience) runs `dobbyengine.EnsureLive(hubgeom.DobbyStart(prime))` from the prime or any beam and prints, per worktree, `started` or `live`, and for a prime another machine owns its owner; it is idempotent.
`dobbyCmd` (hidden, internal audience, launched only by `SpawnDetached`) takes the worktree root, resolves the prime, registers itself through `registerDobby`, which writes this process's `runproc.SelfRecord(runproc.RoleDobby)` under the geometry's beam slug, or `runproc.HubOwner` for the prime, so the pair teardown of a landed beam ends that beam's dobby, and runs `dobbyengine.Run` over `hubDobbyDeps` under a SIGINT/SIGTERM context, printing `already_running` when the lock is held.
Bound: `runproc.End` of the beam's own slug, from a cancel or an owner pause of the beam's landing run, also ends its dobby; by then `BeamLandRefusal` has left the beam no members, so it drives nothing, and `lyx beam start` brings it back if the beam stays.
`hubDobbyDeps` fills every engine seam over the hub, from the hub root and never from inside a worktree: the local batten status files and each run's parent through `hubgeom.ParentOf`, the board and owners through `boardengine`, the pause record through `pauserecord`, the watches through `internal/fswatch`, the pass spawn of `lyx batten run`, the owner pause and lock release through `tendercli` and `parentlock`, the notice through `orchcli.NotifyFromScoped(worktree, slug, dobbyProgram, line)` so the sender is `ly:DOBBY` for the prime and `ly:<slug>:DOBBY` for a beam, `beam.yaml` through `dobbyengine.LoadConfig`, the binary check through `selfexec`, the tidy agent's shuttle runner from the worktree's own reed geometry with `sonnet[medium]` resolved through the model registry, and the stencils dir.
`create` and `promote` start the new beam's dobby as their last step through a `StartDobby` seam (`dobbyengine.SpawnDetached` of its geometry when not live), reporting `dobby_started`; a failure is a step failure naming `lyx beam start`.
`PATTERN.md` gains, in its Shed and loom section:

```
- `PATTERN-dobby-starts-runs` — Starting or driving a board task's run: its parent's dobby, from an empty status, in batten passes; a hand `lyx batten run` is one pass.
```

`beamcli/doc.go` describes the start and dobby verbs and drops "it has no engine package of its own"; `docs/overview.md`'s beam module line names `lyx beam start` and dobby, with "umbrella" read as "beam" in every text this card touches.
Tests: `TestCreate` and `TestPromote` gain rows asserting the dobby start is the last step and a failed start names `lyx beam start`; `TestRegisterDobby` (tier0, temp runproc dir, parallel) rows: a beam geometry's record lists under the beam's slug with role `dobby`, and a prime geometry's under `runproc.HubOwner`, so card 3's `End` row over the beam's slug reaches it; the held lock's exit is pinned by card 32's `TestRun_LockOwnerAndWakeups`.
`lyx beam start` runs from a beam through `runVerb`, which resolves the prime as `create`'s path does, so it carries no beam row of its own.

**ImpactSummary:** The beam module gains two verbs and its daemon wiring; create and promote gain one last step behind a seam their tests fake.
