# Card 4 — The Go driver, the step loops and a seeded batten register themselves under their run's slug

**Create:**
- `plan:internal/loomcli#registerRunProcess` -> `func registerRunProcess(loc *lyxcwd.Location, owner, role string)`
- `plan:internal/loomcli#TestRegisterRunProcess` -> `func TestRegisterRunProcess(t *testing.T)`
- `plan:internal/battencli#battenCLI.registerSeededRun` -> `func (c *battenCLI) registerSeededRun(location *lyxcwd.Location, runID, verb string)`
- `plan:internal/battencli#TestBattenLoopSpec_OnStartRegistersTheLoop` -> `func TestBattenLoopSpec_OnStartRegistersTheLoop(t *testing.T)`

**Edit:**
- `internal/shedverbs#LoopSpec`
- `internal/shedverbs#runLoop`
- `internal/shedverbs#TestLoop_RunsUntilStopAndPrintsOneEnvelope`
- `internal/shedverbs#TestLoop_RefusalsBeforeAnyStep`
- `internal/loomcli#loopSpecFor`
- `internal/loomcli#loomCLI.driveCmd`
- `internal/battencli#battenLoopSpec`
- `internal/battencli#battenCLI.armSeed`
- `internal/battencli#ResumeInWindow`
- `internal/battencli#TestArmSeed_RunAndStepSeedBeforeWire`
- `internal/battencli#TestArmSeed_OwnDriverLLMRefusesTheSameWayOnASeededRun`
- `internal/shedverbs/doc.go`
- `internal/loomcli/doc.go`

**Uses:**
- `internal/runproc#Register`
- `internal/runproc#SelfRecord`
- `internal/runproc#List`
- `internal/runproc#RoleDriver`
- `internal/runproc#RoleLoop`
- `internal/runproc#RoleBatten`
- `internal/fabricengine#RunProcDir`

**Intent:**
D2's self-registration: a long-lived run role registers itself, so every launch path of it is covered whoever started it.
The loop's step child stays recorded in the loop's pid file and tied to the loop, as today.

`shedverbs.LoopSpec` gains `OnStart func()`, documented beside `OnExit`: `runLoop` calls it once, after the pid file is written and the start is logged and before the first step; a loop that refuses before its first step never calls it; nil is skipped.
shedverbs takes the hook rather than importing `runproc`, so its import allowlist is unchanged.
`TestLoop_RunsUntilStopAndPrintsOneEnvelope` sets `OnStart` and asserts it ran exactly once, before the first child step; `TestLoop_RefusalsBeforeAnyStep` sets it on every row and asserts it never ran.

`registerRunProcess` registers the calling process with `runproc.SelfRecord(role)` under `owner` in `fabricengine.RunProcDir(loc.HubPath)` and logs a failure at Warn, never failing the role.
`loopSpecFor` sets `OnStart` to register `runproc.RoleLoop` under `loc.WorktreeName`, the task pair's slug.
`driveCmd`'s `RunE` registers `runproc.RoleDriver` under `c.location.WorktreeName` on start, before `sheddriver.DriveVerb` takes the driver lock; a second driver the lock refuses exits at once, and its record is pruned at the next read (D1), so registering ahead of the lock admits nothing.
`TestRegisterRunProcess` builds a location over a temp hub, calls `loopSpecFor(...).OnStart()` and `registerRunProcess(loc, slug, runproc.RoleDriver)`, and asserts `runproc.List` returns the test's own pid under the slug with roles `loop` and `driver`;
a location whose `RunProcDir` path is taken by a plain file registers nothing and logs one Warn (`internal/testkit/logcapture`).

`battenLoopSpec` sets `OnStart` to register `runproc.RoleLoop` under its slug, through a batten-side call of `runproc.Register` with `fabricengine.RunProcDir(location.HubPath)`.
`TestBattenLoopSpec_OnStartRegistersTheLoop` calls `battenLoopSpec(...).OnStart()` over a temp hub and asserts a `loop` record of the test's pid under the slug.

`registerSeededRun` registers `runproc.RoleBatten` under `runID` only for verb `run` without `--window` (`c.windowFlag`), since a window opener only opens the window and exits.
`armSeed` calls it on each successful return, a found agreeing seed or a freshly written one, so a starting batten refused before or at the seed registers nothing (D2, D8).
`ResumeInWindow`, `lyx adopt`'s step 5, runs the `--window` body in the adopting process, so it sets `c.windowFlag` before `armAt`: the adopting process is never registered as a batten, and only the in-window run it opens registers.
`TestArmSeed_RunAndStepSeedBeforeWire` gains assertions in its verb loop: after `armSeed` succeeds, `run` leaves one `batten` record of the test's pid under the slug in the temp hub's `RunProcDir`, and `step` leaves none; a `run` row with `windowFlag` set leaves none.
`TestArmSeed_OwnDriverLLMRefusesTheSameWayOnASeededRun` gains the assertion that the refused `armSeed` leaves no record.

The `shedverbs` and `loomcli` package docs state the hook and which role registers where; `battencli` keeps its package comment in `cli.go`, so the doc comments of `battenLoopSpec`, `armSeed` and `ResumeInWindow` state its registrations.

**ImpactSummary:** one new LoopSpec hook, registration call sites in loomcli and battencli, and ResumeInWindow sets the window flag it already runs under; no signature changes.

**Commit:** 4: the Go driver, the step loops and a seeded batten register their run processes
