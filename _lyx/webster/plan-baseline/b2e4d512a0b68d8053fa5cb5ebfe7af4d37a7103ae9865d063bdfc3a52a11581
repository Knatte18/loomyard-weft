# Card 23 — wake-reconcile

**Create:**
- `plan:internal/hibernateengine#WakeReconcile` -> `type WakeReconcile struct`
- `plan:internal/hibernateengine#WakeRun` -> `type WakeRun struct`
- `plan:internal/hibernateengine#reconcileAtWake` -> `func reconcileAtWake(deps Deps, r Request) (lift bool, notice string, err error)`
- `plan:internal/hibernatecli#wakeReconcile` -> `func wakeReconcile(location *lyxcwd.Location) (hibernateengine.WakeReconcile, error)`
- `plan:internal/hibernatecli#wakeReconcileFromReport` -> `func wakeReconcileFromReport(report fabricengine.OriginReport) hibernateengine.WakeReconcile`
- `plan:internal/hibernatecli#TestWakeReconcileFromReport` -> `func TestWakeReconcileFromReport(t *testing.T)`

**Edit:**
- `internal/hibernateengine#Deps`
- `internal/hibernateengine#Watch`
- `internal/hibernateengine/doc.go`
- `internal/hibernatecli#hubDeps`
- `internal/hibernateengine#newFakeWorld`
- `internal/hibernateengine#TestWatch`
- `internal/hibernatecli#TestHibernateVerbs_WireTheHub`

**Uses:**
- `plan:internal/fabricengine#ReconcileOrigin`
- `plan:internal/fabricengine#ReconcileOptions`
- `plan:internal/fabricengine#OriginReport`
- `plan:internal/pauserecord#SetGateExempt`
- `plan:internal/pairteardown#HubSeamOptions`
- `plan:internal/pairteardown#Teardown.OriginSeams`

**Intent:** D8's wake: the hibernate daemon runs the hub reconcile after the wake and before it lifts its pause entry, through its `Deps` seams as it reaches tender today.
`Deps` gains `Reconcile func() (WakeReconcile, error)` and `ExemptNetwork func(ref tenderengine.EntryRef) (restore func())`; the engine imports no fabricengine, so `WakeReconcile` is its own view of the report: unreachable, the D3 stop with its way forward, one `WakeRun` per stopped or ended entry (slug, stopped or ended, the stop's way forward, the busy flag, whether a parked push holds its weft's push lock, whether its stop is a branch gone on origin, landed or removed), and `Mutations []string`, one rendered line per entry of the report's mutation record.
`hibernatecli#hubDeps` fills `Reconcile` with `wakeReconcile` (`ReconcileOrigin` over the hub scope, seams from `pairteardown.OriginSeams` with `HubSeamOptions`, options of a 2-minute bound, a 10-second interval and a 20-second per-attempt deadline), which maps the report through the pure `wakeReconcileFromReport`: a `WakeRun`'s parked-push flag taken from its busy weft's `BranchOutcome.PushLockHeld`, and its stop's way forward, like the D3 stop's, from the stopped branch's `BranchOutcome.WayForward`, never composed here, and each mutation entry rendered as its kind, target and detail on one line.
`hubDeps` fills `ExemptNetwork` with `pauserecord.SetGateExempt` of the entry's scope and pause time.

`Watch`, after the wake and the move to resuming, calls `reconcileAtWake` in place of the plain lift:
- It reads the live entries (`TenderStatus`). When a hub entry the request does not own is live (the request found an operator's hub pause and owns none, or its owned entry was lifted and another added), it skips the reconcile without waiting at the gate, lifts only an entry the request owns, and the notice names the standing entry, `lyx resume` as its way forward, and that `lyx orch start` or `lyx cleanup --apply` runs the reconcile after that lift. With no hub entry live at all it reconciles with nothing to exempt.
- Otherwise it exempts the owned entry for the reconcile's duration (`ExemptNetwork`, restored when `Reconcile` returns), so only the reconcile's ls-remote calls, their retries and its fetches pass, every other process and entry staying held.
- The daemon has no envelope, so it logs each of the returned `WakeReconcile.Mutations` lines at Info through `internal/logger`, and they never enter the notice.
- Unreachable through the bound: nothing moved, the owned entry is lifted as today, and the notice names `unreachable` and `lyx cleanup --apply` as the reconcile to run once the network is back.
- A D3 stop keeps the hub entry live, and the notice names the stop and `lyx resume` as the way forward once settled.
- Otherwise each stopped entry gets a plain pair pause entry through `Pause` with its slug (not recorded as owned, so it outlives the request and the hub entry's lift), and so does each ended entry with a busy flag; the owned entry is then lifted; the notice names, per stopped run, its stop's way forward then `lyx resume <slug>` (only `lyx resume <slug>` for a pair busy on a parked push, and `lyx cleanup --apply` in place of `lyx resume <slug>` for a pair stopped as gone, which no resume can carry on), per busy ended entry the landing or removal and `lyx cleanup --apply` (never `lyx resume <slug>`), and lists the ended entries, leaving out one with no local pair (card 17).
- A `Reconcile` error is handled as unreachable is: nothing further moves, the owned entry is lifted, and the notice names the error and `lyx cleanup --apply` as the reconcile to run.
- A `TenderStatus` error before the reconcile fails the request through the existing `fail` path, which lifts the owned entry and queues its one notice.
- A `Pause` error while adding a pair pause entry leaves that pair unheld, the remaining entries still get theirs and the owned entry is still lifted; the notice names each pair left unheld and `lyx tender pause <slug>` as the way to hold it.
The notice stays one line, as the queue requires.
Bound: the exemption passes only the reconcile's own network calls and only the owned entry, and ends with the reconcile; an entry the request does not own is never passed.
`doc.go`'s wake paragraph is rewritten to this sequence.

`TestWatch` gains rows over `newFakeWorld`'s new fake `Reconcile` and `ExemptNetwork`: wake with the hub pause live reconciles under the exemption, restores it and lifts the hub entry; wake under an operator's hub pause skips the reconcile without exempting, moves nothing, leaves that entry live and names it and `lyx resume`; a parked push holding a paused pair's push lock reports that pair busy, gives it a pair pause entry naming `lyx resume <slug>` and lifts the hub entry; a paused run whose weft could not move gets a pair pause entry that outlives the request, the queued notice containing the fake `WakeRun`'s exact way-forward line followed by `lyx resume <slug>`; a pair stopped as gone gets a pair pause entry, the notice containing the fake `WakeRun`'s exact way-forward line followed by `lyx cleanup --apply`, with no `lyx resume <slug>`; a D3 stop keeps the hub entry live, the notice containing the fake D3 stop's exact way-forward line followed by `lyx resume`; an ended busy entry gets a pair pause entry and the notice contains its landing and `lyx cleanup --apply`, with no `lyx resume <slug>`; a fake `WakeReconcile` carrying two `Mutations` lines logs each at Info and leaves both out of the notice, a serial subtest `TestWatch` runs before its parallel table loop, since it captures the process-global logger output through `logger.SetOutput` and raises the process-global level to Info through `logger.SetVerbosity(1)`, restoring both; unreachable through the bound lifts the entry and names `unreachable` and `lyx cleanup --apply`; with no hub entry live at all the reconcile runs and `ExemptNetwork` is never called; a `Reconcile` error lifts the owned entry and names the error and `lyx cleanup --apply`; a `TenderStatus` error fails the request with the owned entry lifted; a `Pause` error for one stopped pair still holds the other stopped pair, lifts the owned entry and names the unheld pair and `lyx tender pause <slug>`.
`TestHibernateVerbs_WireTheHub` asserts the two new seams are filled.
`TestWakeReconcileFromReport` (untagged) builds each `OriginReport` by hand, with rows: a stopped branch's `WayForward` reaches its entry's `WakeRun` exactly; a busy weft's `PushLockHeld` becomes the `WakeRun`'s parked-push flag; a branch stopped as `gone` sets the `WakeRun`'s gone flag; a D3 stop on Bolt carries Bolt's `WayForward` as the D3 stop's line; a report whose record holds a teardown's removal and its snapshot tag yields one `Mutations` line per entry.
The retry within the bound and its success once origin answers are reached by card 15's ls-remote retry with injected options.

**ImpactSummary:** Two new `Deps` seams, filled only by hibernatecli through a pure report mapping; the wake's lift path changes, every other phase of `Watch` is unchanged.

**Commit:** 23: hibernate wake runs the hub's origin reconcile before the lift
