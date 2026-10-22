# Card 12 — tender's and creel's runners read the hub's pause, and creel holds a paused inject

**Create:**
- `plan:internal/tendercli#TestNewShuttlePair_TellsTheHubsPauseGeometry` -> `func TestNewShuttlePair_TellsTheHubsPauseGeometry(t *testing.T)`
- `plan:internal/orchcli#TestCLI_OrchRunnerReadsNoPause` -> `func TestCLI_OrchRunnerReadsNoPause(t *testing.T)`

**Edit:**
- `internal/tendercli#newShuttlePair`
- `internal/creelcli#wireShuttle`
- `internal/creelcli#mapDeliveryError`
- `internal/creelcli#TestHubDelivery`

**Uses:**
- `internal/shuttleengine#ErrPaused`
- `internal/shuttleengine#Runner.SetPause`
- `internal/creelcli#newStubbedDelivery`
- `internal/creelcli#stubOps`

**Intent:**
D1's geometry for the two hub-level runners that type a line and are told none today, and D2's creel delivery.
`newShuttlePair` and `wireShuttle` each resolve `hubgeom.PauseGeometry` for their location and tell their runner, through `SetPause`, its record path with an empty slug: the hub's geometry, so the runner's own fallback slug is the prime's, and the target's own name decides the slug every send judges.
Both fallbacks of card 9 can therefore let a line into a session that only a slug entry covers when the target's name cannot be read, the accepted residual D1 names.
The orch runner (`orchcli`) stays told no geometry.
`mapDeliveryError` maps an error wrapping `shuttleengine.ErrPaused` onto `creelengine.ErrBusy`, so the delivery worker requeues the inject untouched and types it after the resume, as creel's own `Paused` check already holds it; the worker's `Paused` reading stays read-only and records nothing.
Its doc comment names the paused mapping.

`TestHubDelivery`'s sentinel table gains a `paused` row mapping onto `creelengine.ErrBusy` for `Send` and `Command`, and a new subtest calls `wireShuttle` on the pair worktree with a stub engine set and asserts the runner's `PauseGeometry` is the hub's record path with an empty slug.
`TestNewShuttlePair_TellsTheHubsPauseGeometry` builds a pair over a `hubforge` hub and asserts the same of the pair's runner; it goes in the package's integration-tagged `cli_test.go`, since `newShuttlePair` resolves a real location's reed and shuttle config and its pause geometry, which no fake supplies, and the untagged `deps_test.go` touches neither reed nor the runner.
`TestCLI_OrchRunnerReadsNoPause` pins the orch side of `lyx orch start`'s acceptance criterion: over a `hubforge` prime, as `TestOrchIntegration_Refusals` builds one, it runs the orch command's pre-run for a verb that spawns nothing and asserts the wired runner's `PauseGeometry` is the zero value, so a `SetPause` added to the orch wiring fails it; card 9's no-geometry row pins that such a runner types under a live hub entry.
It sits in an integration-tagged test file of `internal/orchcli`.

**ImpactSummary:** two runner constructions gain a pause geometry, which only makes their sends refuse covered targets; creel's error mapping gains one row; no exported surface changes.
