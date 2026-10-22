# Card 16 — batten-pre-drive-check

**Create:**
- `plan:internal/battencli#battenCLI.preDriveCheck` -> `func (c *battenCLI) preDriveCheck() error`
- `plan:internal/battencli#TestPreDriveCheck` -> `func TestPreDriveCheck(t *testing.T)`

**Edit:**
- `internal/battencli#battenCLI`
- `internal/battencli#battenCLI.armAt`
- `internal/battencli#battenCLI.battenPreRun`
- `internal/battencli#battenCLI.battenPreStepUnmarked`
- `internal/battencli#battenCLI.specFor`

**Uses:**
- `internal/battencli#newFakeReceiver`
- `plan:internal/battencli#takeCancelMarker`
- `internal/shedengine#Shed`

**Intent:**
Install D3's second gate, which re-reads the cancel marker and the seed at the two points a batten that passed `armAt` before the marker was written still reaches.

`preDriveCheck` reads `c.guard.Marker`: a present marker refuses with `cancelMarkerRefusal`; a nil `Marker` reads no marker, as in `startGuard.refuse`, and the seed read below still runs.
It then reads the seed (`shedrun.ReadSeed(c.location, c.slug)`): absent refuses with `battencli: <slug> has no seed, so its run was cancelled or never started; run "lyx batten run <slug>" for a fresh start`.
It writes nothing either way.
`battenCLI` gains the field `preDrive func() error`, which `armAt` sets to `c.preDriveCheck`; nil checks nothing, which only a hand-populated receiver has, so `newFakeReceiver`'s existing tests, whose location holds no seed, run as before.

The first call site: in `battenPreRun` and `battenPreStepUnmarked`, the seeding of an absent status moves wholly into the `state.UpdateJSON` closure, under the status lock: the closure leaves a found status untouched as today, otherwise calls `c.preDrive` (refusing with its error and writing nothing), then `battenshed.ClearRunMarkers(c.env.ScratchDir)`, then returns the Worktree-Create running status; so a refused seeding writes no status and clears no marker.
`battenPreStepUnmarked` maps the closure's refusal to `shedverbs.KindBusy`, a failed marker clear and a failed write staying `KindUnseeded`: the run is held by a cancel or no longer seeded, so the Go driver's `KindBusy` row exits and leaves it to its holder instead of spawning a tackler (`decideDrive`'s `KindBusy` case ahead of the tackler rows).
The second call site: `specFor`'s `BuildShed` sets the built shed's `LockedCheck` to `c.preDrive`, so batten's shed `Run` and `Step` call it once they hold `shedrun.RunLock` and before they read or write the status, for a resume whose status was already present; the step verb reports its refusal as batten's `StepBusyKind`, `KindBusy`, with the check's own message (card 6), so the driver exits there too.

`TestPreDriveCheck` uses `newFakeReceiver` with `preDrive` set to `preDriveCheck` and a guard reading the real marker under the receiver's location: with a seed written and no marker, `battenPreStep` seeds as `TestBattenPreStep_SeedsAbsentStatus` expects; with a marker taken by `takeCancelMarker`, `battenPreStep` and `battenPreRun` refuse, `battenPreStep` with kind `KindBusy`, write no status file and leave a planted run-marker file in place; with the marker released and no seed, both refuse naming `lyx batten run some-slug` with no status written; the shed `BuildShed` returns, run over a present status with the marker taken, refuses without calling a producer and leaves the status bytes unchanged.

**ImpactSummary:** Both pre-run hooks and `BuildShed` are reached only through batten's spec; with `preDrive` nil every existing path behaves as before.
