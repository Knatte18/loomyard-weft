# Card 40 — teardown-dobby-liveness

**Edit:**
- `internal/pairteardown#Teardown`
- `internal/pairteardown#New`
- `internal/pairteardown#Teardown.registeredBattenAlive`
- `internal/pairteardown#Teardown.endOwnedProcesses` -> `func (t *Teardown) endOwnedProcesses(slug string, spare ...string) ([]runproc.Ended, error)`
- `internal/pairteardown#Teardown.EndBeforeSessionDown`
- `internal/pairteardown#fakeSubstrates`
- `internal/pairteardown#fakeSubstrates.teardown`
- `internal/pairteardown#TestEndBeforeSessionDown`
- `internal/pairteardown/doc.go`

**Create:**
- `plan:internal/pairteardown#TestRegisteredBattenAlive_ReadsADobbyDrivenRun` -> `func TestRegisteredBattenAlive_ReadsADobbyDrivenRun(t *testing.T)`

**Uses:**
- `internal/runproc#RoleDobby`
- `internal/runproc#EndSparing`

**Intent:**
Read the gap between a run's passes as a live lifecycle owner, and keep a beam's dobby through its session's end (D1, D4).
`registeredBattenAlive` reads a run as having a live owner when a registered batten of the slug lives, as today, or when a live `RoleDobby` record is registered under the run's parent (the parent beam's slug, or `runproc.HubOwner` for the prime), batten's status reads `running` and the slug's cancel marker (`shedrun.CancelMarker`) is absent: the runs dobby passes.
It reads those facts through `Teardown` seams `New` fills, so a fake drives each: batten's state through the existing `battenRun`, the parent through a new `parentOf` (`hubgeom.ParentOf` over the prime), the owner's live records through the existing `ownedProcesses` reading under that owner, and the marker through a new `cancelMarked` (a stat of `shedrun.CancelMarker`).
The `endOwned` seam takes the roles to spare, `func(slug string, spare ...string) ([]runproc.Ended, error)`, and `endOwnedProcesses` passes them to `runproc.EndSparing`.
So every reader of it, the claim settlement, the session end and `driverChecksSkipped`, never reads the gap between passes as a gone batten, and still reads a halted run as settled, since a halted batten status is not `running`.
`EndBeforeSessionDown`, the end `lyx reed down` runs through `internal/reedcli`, ends the slug's registered processes through `runproc.EndSparing` with `RoleDobby` spared, so a beam's dobby, which runs from the hub root outside the session and whose member runs still need it, stays live; a teardown's own end is unchanged and ends a landed beam's dobby.
The package doc states both.
Tests: `fakeSubstrates` gains the parent, the cancel mark and the spared roles its `endOwned` was called with, and `fakeSubstrates.teardown` wires them; `TestRegisteredBattenAlive_ReadsADobbyDrivenRun` (tier0, over `fakeSubstrates`, parallel) rows: a running batten status with a live dobby record under the parent reads alive; a halted status reads not alive; a cancel marker reads not alive; no dobby record reads not alive. `TestEndBeforeSessionDown` gains a row asserting its `endOwned` call spares `RoleDobby`.

**ImpactSummary:** One liveness read widens for dobby-driven runs through two new Teardown seams, and the end seam gains spared roles that only the session end passes; its three readers inherit the change.
