# Card 10 — `lyx loom start` resumes a parked driver

**Edit:**
- `internal/loomcli/start.go`
- `internal/loomcli/driverlaunch.go`
- `internal/loomcli/cli.go`
- `internal/loomcli/wiring.go`
- `internal/loomcli/start_driver_test.go`

**Create:**
- `internal/loomcli/driverresume.go`
- `internal/loomcli/driverresume_test.go`

**Prosa:**
- `docs/overview.md`

**Uses:**
- `plan:internal/shedrun#ParkMarker`
- `plan:internal/loomcli#driverResumeLine`
- `internal/loomcli#driverReportPath`
- `internal/loomcli#resolveDriverStrandAction`
- `internal/shuttleengine#Runner.Send`

**Intent:**
Make `lyx loom start` the resume route (decision record D8), so the hub's `lyx loom approve` then `lyx loom start` and batten's `Spawn` (`lyx loom start --no-attach`) keep their commands and never need to know a driver parked.

Seams (`driverlaunch.go`, `cli.go`, `wiring.go`):
- A `driverSender` interface with `SendDriver(guid, text string) error`, implemented by `runnerDriverStarter` through `shuttleengine.Runner.Send`, which verifies delivery and refuses a pane whose provider is not input-ready.
- `loomCLI` gains a `driverSender` field and a `driverResumeWait func()` field (the pause between send attempts), both filled in `wiring.go` beside `driverStarter`.

`driverresume.go` holds `(*loomCLI).resumeParkedDriver(guid string) error`:
1. Compose a fresh report path with `driverReportPath` for the resolved run-id, and create its parent directory.
2. Build the line with `driverResumeLine`.
3. Call `SendDriver` up to a declared attempt cap (`driverResumeSendAttempts`, about twenty), running `driverResumeWait` between attempts (production: a declared interval of about three seconds, so the bounded wait is about a minute).
   Every attempt is logged, and the outcome at the end, per Live-Substrate Spawn Observability; the loop caps the attempt count, not only elapsed time.
4. On a verified send, remove the marker at `shedrun.ParkMarker`, treating an already-absent marker as success (the driver may have removed it first), and log the resume with the report path.
5. When every attempt fails, return an error naming the attempts, the last send error, and the retry (`run "lyx loom start" again once the driver's pane is ready`), leaving the marker in place.

`runDriverSpawnAndWait` (`start.go`) branches after it resolves the driver strand action, before any spawn:
- Spawn needed (`mustSpawnDriver` true, a dead or absent driver): remove a stale marker if present, then spawn exactly as today.
  A removal failure other than not-exist releases the bootstrap lock and refuses on the envelope.
- Driver strand live and the marker present: call `resumeParkedDriver`, spawn nothing.
  An error releases the bootstrap lock and refuses on the envelope; success returns true, so the handover runs as today (the envelope under `--no-attach`, otherwise the attach).
- Driver strand live and no marker: send nothing, spawn nothing, hand over as today.
The branch reads only strand liveness and the marker, never the seed's driver field a second time, per the Driver Choice Single-Site Invariant.

`startCmd`'s `Long` help (step 3 and the `--no-attach` paragraph) and the `--no-attach` flag text describe the three cases: resume a parked ly-drive driver by typing one line into its pane, and for `--no-attach` return once that line's delivery is verified; refuse after the bounded wait when the pane is not ready, leaving the driver parked.
`docs/overview.md`'s `start` description in the loom module entry and its "the bootstrap" entry say the same in one sentence each.

Tests (untagged, fakes only, no real pane and no real sleep):
- `driverresume_test.go`: a fake sender that succeeds records exactly one call whose text is a single line naming a path under the run's drive-reports directory, and the marker file is gone afterwards.
- A sender failing twice then succeeding makes three calls with two waits.
- A sender that always fails makes exactly `driverResumeSendAttempts` calls; the error names a retry, and the marker file is still on disk.
- `start_driver_test.go`, through `runDriverSpawnAndWait` with `newTestSpawnAndWaitReceiver` extended for the sender:
  - a live driver strand with the marker present sends once, removes the marker, and never calls the starter;
  - a live strand without the marker neither sends nor starts;
  - a dead strand with a stale marker removes the marker and starts a fresh driver;
  - a live strand with the marker and a sender that never succeeds refuses, releases the bootstrap lock, keeps the marker and never starts.

**ImpactSummary:** `lyx loom start` gains a resume branch taken only when a live driver strand and the park marker coexist, and removes a stale marker before a spawn; with no marker on disk, every existing path, including batten's `Spawn`, behaves exactly as before.

**Commit:** 10: loomcli: lyx loom start resumes a parked ly-drive driver
