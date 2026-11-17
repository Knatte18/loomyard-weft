# Card 15 — tender-bolt-hold

**Create:**
- `plan:internal/tenderengine#BoltHoldState` -> `type BoltHoldState struct`
- `plan:internal/tenderengine#BoltHoldRequest` -> `type BoltHoldRequest struct`
- `plan:internal/tenderengine#BoltHoldReport` -> `type BoltHoldReport struct`
- `plan:internal/tenderengine#SyncBoltHold` -> `func SyncBoltHold(deps Deps, req BoltHoldRequest) (BoltHoldReport, error)`
- `plan:internal/tenderengine#TestSyncBoltHold` -> `func TestSyncBoltHold(t *testing.T)`

**Edit:**
- `internal/tenderengine#PauseRequest`
- `internal/tenderengine#Pause`
- `internal/tenderengine#Deps`
- `internal/tenderengine#Resume`
- `internal/tenderengine#liftEntries`
- `internal/tenderengine#TestResume`
- `internal/tendercli#HubDeps`
- `internal/tendercli#resumeAside`
- `internal/tendercli#TestPauseAndResume_WriteLiftAndRefuseOverTheHub`
- `internal/tenderengine/doc.go`
- `internal/tendercli/doc.go`

**Uses:**
- `plan:internal/pauserecord#BoltHoldCaller`
- `internal/boardengine#ReadOwners`
- `internal/boardengine#PullBolt`

**Intent:** Write and drop the `bolt-hold` slug entries, and keep `lym tender resume` from lifting one while the Bolt hold stands (D5).

`PauseRequest` gains `HoldMachine`, which `Pause` writes as the entry's `Machine`.
`SyncBoltHold(deps, req)` settles one dobby's runs against the hold: `BoltHoldRequest` holds `Machine`, `Standing`, `Hold` (the slugs the act rule answers `wait` with reason `held` for) and `Scope` (every slug the caller drives).
While `Standing`, each slug of `Hold` with no live entry of its own scope gets a `bolt-hold` entry through `Pause` (the caller's grace, `BoltHoldCaller`, `HoldMachine`), so its sessions and network calls are held; a slug with a live entry gets none.
Each `bolt-hold` entry of a `Scope` slug outside `Hold`, and every one in `Scope` once `Standing` is false, is lifted through `liftEntries` by its exact ref, so only the `bolt-hold` caller's entries go and an operator's hub entry or slug entry stands.
`BoltHoldRequest` also holds `HoldsParentLock func(slug string) (bool, error)`: a slug of `Hold` that holds its parent lock gets no entry, and its `bolt-hold` entry is lifted, so the gate does not hold its step in flight while the act rule still starts no producer for it (D5);
a failed read of one slug is joined into the error and leaves that slug's entries as they are, and a nil seam reads no slug as holding its lock.
A run whose push turns owed answers `push_owed`, not `held`, so its entry is dropped and its push passes the gate.
The fix task never reads `held`, so it gets none.
No hub entry is ever written.

`Deps` gains `BoltHold func() (BoltHoldState, error)`: `Standing`, `Machine` and `FixTask` of the hold the local Bolt holds; a nil seam reads no hold.
`Resume` judges it:
while a hold stands, a hub-wide resume leaves every `bolt-hold` entry standing;
`lym tender resume <slug>` of a slug only a `bolt-hold` entry covers refuses with nothing lifted and nothing cleared, judged after the verdict and before `req.Clear`, so a refused resume leaves the run's pause intent standing and pushes nothing: `tender: <slug> is held by an adopt on machine <m>; way forward: wait for that adopt's hold to clear, and the worktree's dobby lifts its entry at its next tick (with no dobby running, run lym beam start); if machine <m> has stopped, ask the operator`;
a `bolt-hold` entry whose hold no longer stands is lifted like any other entry, so a stopped dobby's leftover entry never strands the slug;
lifting a slug's own entry of another caller under a standing hold writes a `bolt-hold` entry in its place, unless the slug is the hold's fix task.
No message names `--take-hold`.
`tendercli`'s `HubDeps` fills `BoltHold` from `boardengine.ReadOwners` of the hub's Bolt after a `boardengine.PullBolt`, an unreachable origin reading the local Bolt, so a resume reads a hold origin cleared as cleared.
`resumeAside` gains `Hold`, so the verdict `hubRunIntents` judges for `lym tender resume` sets the hold aside and `Resume`'s own hold rules decide;
without it the act rule's held refusal (card 13) refuses every resume under a hold before any lift.

Tests, tier0 unless their file names another:
`TestSyncBoltHold` covers: entries written for held slugs, none for a slug with a live entry, none for an unheld slug; entries dropped for a slug that left `Hold` and for all once the hold no longer stands, with a hub entry and an operator's slug entry left standing; a held slug holding its parent lock gets no entry and its standing `bolt-hold` entry is lifted.
`TestResume` gains rows for the three resume rules, the refusal message, a refused resume of a `bolt-hold`-only slug whose run carries a pause intent leaving `Clear` uncalled and the intent standing, and the lift of a `bolt-hold` entry once no hold stands.
`TestPauseAndResume_WriteLiftAndRefuseOverTheHub`, tier1, gains a row over the hub: under a standing Bolt hold, `lym tender resume <slug>` of a slug with its own pause entry lifts that entry and writes a `bolt-hold` entry in its place.
Both docs describe the caller's writer and the resume rules.

**ImpactSummary:** One new engine function and one new Deps seam; Pause gains an optional field and Resume three hold rules, with tendercli's HubDeps the only production filler; the resume verb sets the hold aside so those rules are reached.
