# Card 2 — driver-network-wait

**Edit:**
- `internal/sheddriver#decideDrive`
- `internal/sheddriver#driveDecision`
- `internal/sheddriver#driveFacts`
- `internal/sheddriver#Deps`
- `internal/sheddriver#goDriver.run`
- `internal/sheddriver#goDriver.facts`
- `internal/sheddriver#goDriver.settleStop`
- `internal/sheddriver#goDriver.handBackStop`
- `internal/sheddriver#TestDecideDrive`
- `internal/sheddriver#TestGoDriver_Settle`
- `internal/sheddriver/doc.go`
- `internal/loomcli#loomCLI.goDriverDepsFor`
- `internal/loomcli/doc.go`
- `internal/battencli#battenCLI.driverDepsFor`
- `internal/loomengine#AnomalyTransientStop`
- `contracts/stencils/friction/friction-template-reflection.md`
- `cmd/lym#unboundedWaitFiles`

**Delete:**
- `internal/sheddriver#networkRestepWaits`
- `internal/sheddriver#networkRestepSpan`
- `internal/sheddriver#goDriver.networkNotice`

**Create:**
- `plan:internal/shedrun#DriveWaitRecord` -> `func DriveWaitRecord(l *lymcwd.Location, runID string) string`
- `plan:internal/sheddriver#DriveWait` -> `type DriveWait struct`
- `plan:internal/sheddriver#networkWait` -> `func networkWait(repairCount int, ceiling time.Duration) time.Duration`

**Uses:**
- `internal/shedrun/doc.go`
- `plan:internal/fabricengine#Config.PushTiming`
- `internal/sheddriver#TestGoDriver_Wait`

**Intent:** D6: the Go driver waits out a network stop instead of handing it back.

`decideDrive` re-steps a stop `networkFailure` reads as `git-transport` or `github-api` with no count cap.
Its wait is `networkWait(facts.RepairCount, facts.NetworkCeiling)`: 15 s, doubled per earlier re-step of the stop key, capped at the ceiling.
The hand-back row "a network failure that outlasted its re-steps" goes, and with it `driveDecision.NetworkNotice`, its branch in `goDriver.handBackStop`, `networkNotice`, `networkRestepWaits` and `networkRestepSpan`.
`driveFacts` gains `NetworkCeiling`, which `goDriver.facts` copies from the new `Deps.NetworkWaitCeiling`.
A zero `NetworkWaitCeiling`, as a hand-built `Deps` leaves it, caps the wait at 15 s, so the wait never grows unbounded.
`agent-start` and `host-resources` keep the loop's one re-step, then the tackler, as today.
A refusal and every other class leave the network row at once, a GitHub rate limit and a 401, 403 or 404 among them, since `shedtransient` gives them no network class.
Bound: a stop that stays network-class is re-stepped for as long as it does; D3's summary shows the wait and `lym batten cancel` ends it.

The wait runs between steps in `goDriver.settleStop`, so the run holds no parent lock and no run lock, only the driver lock it already holds.
Before the first wait of a network stop `settleStop` writes a `DriveWait` record, holding the producer, the transient class and the time that first wait started, at the new `Deps.WaitRecordPath`, through `internal/state`.
The record stands across every later wait and the re-steps between them while the stop stays network-class, and keeps its first start time, so the run reads as waiting for the whole wait and D3's threshold counts from its start.
`settleStop` removes it once it settles a stop that is no longer network-class, a done stop and a hand-back among them, and when the context ends a wait.
`goDriver.run` removes any record standing at its start, before its first step, so a record a killed driver left never reads as a live wait under the next driver's lock.
An empty `WaitRecordPath` writes and removes none.
`shedrun.DriveWaitRecord` names the record's file under `shedrun.ScratchDir` beside `shedrun.DriverLock`.
`goDriverDepsFor` sets `WaitRecordPath` to it and `NetworkWaitCeiling` to the hub's `fabric.push_ceiling_s` through `Config.PushTiming`, logging each floor line once.
`battenCLI.driverDepsFor`, batten's own driver in the prime, sets both the same way, `WaitRecordPath` to `shedrun.DriveWaitRecord` over the prime's location and the slug,
so a Land `git-transport` stop (card 16) is waited out under a record its readers see (card 3, card 20).

`cmd/lym#unboundedWaitFiles` gains `internal/sheddriver/drive.go`, the driver's network wait.

The doc comment of `AnomalyTransientStop`, the paragraph of `internal/loomcli/doc.go` that mirrors it, and the reflection template's line 53 say that only `git-transport` and `github-api` stops are re-stepped with waits, for as long as they stay network-class, and that every other class goes to the tackler (finding F2 of `transient-stop-not-failed`).
`internal/sheddriver/doc.go`'s paragraph on the network-failure waits says the same and drops "only a failure that outlasts the waits reaches the parent, as a notice".

Tests, tier0 rows adding under a second to one tier0 run.
`TestDecideDrive`, red-first: a `git-transport` row with `RepairCount` 4 and a 300 s ceiling re-steps with a 240 s wait, a row with `RepairCount` 6 re-steps with a 300 s wait, where the merge base hands back;
a `github-api` row at `RepairCount` 0 waits 15 s, and a row with a zero ceiling waits 15 s.
The rows that asserted the hand-back and its network notice at `RepairCount` 3 are replaced by these.
`TestGoDriver_Settle`'s network rows step four network failures and then a done stop, assert the waits `15s, 30s, 1m, 2m`,
assert the wait record names the producer and the class and keeps its first start time through every wait and re-step, is gone after the done stop, and that no hand-back happens.
The row "a wait the context ends" keeps its error and asserts the record is removed.
A new row starts the driver with a record already standing and asserts it is gone before the first step.

**ImpactSummary:** Changes one rule of the driver's table and its wait; the hand-back of a network stop and its notice go, which only the driver and its tests read.
