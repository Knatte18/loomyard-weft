# Card 19 — orch-start-reconcile

**Create:**
- `plan:internal/orchcli#reconcileHub` -> `func reconcileHub(location *lyxcwd.Location) (fabricengine.OriginReport, error)`
- `plan:internal/orchcli#TestStart_ReconcilesTheHubFirst` -> `func TestStart_ReconcilesTheHubFirst(t *testing.T)`

**Edit:**
- `internal/orchcli#orchCLI`
- `internal/orchcli#Command`
- `internal/orchcli#orchCLI.startCmd`
- `internal/orchcli/doc.go`
- `internal/orchcli#newStartHarness`

**Uses:**
- `plan:internal/fabricengine#ReconcileOrigin`
- `plan:internal/fabricengine#FailedPushFields`
- `plan:internal/pairteardown#HubSeamOptions`
- `plan:internal/pairteardown#Teardown.OriginSeams`

**Intent:** D8's orch start: `lyx orch start` runs the hub reconcile before launching the session, and never refuses on it.
`orchCLI` gains a `reconcile func() (fabricengine.OriginReport, error)` seam, which `Command`'s pre-run wires to `reconcileHub`: `fabricengine.ReconcileOrigin` over the hub scope with the seams of `pairteardown.New(prime).OriginSeams(pairteardown.HubSeamOptions(prime))` and zero options.
`startCmd` calls it after the start lock and before the strand decision; the envelope gains `reconcile` (the report) and `failed_pushes` (`FailedPushFields`), and a fresh launch appends one line to the start prompt summarising both (`OriginReport.Summary` and the count of outstanding failed pushes with their branches), the orch's start note.
An `Unreachable` report is a `warning` on the envelope; a reconcile error, a D3 stop and every entry stop are reported, never a refusal, since the orch is the session that resolves a stop: a run already driving is reported live, or stopped when origin moved its branch.
Under an operator's live hub pause the reconcile's network calls wait at the network gate until the lift, as every other network call of the hub does; the gate's wait line already names the pause.
The long help names the reconcile and the two envelope keys, and `doc.go` the start sequence.
Bound: the start never refuses on the report, so a stop reaches the orch as a line to act on.

`TestStart_ReconcilesTheHubFirst` (untagged, the existing start fakes plus a fake `reconcile`) rows: the reconcile runs before the strand is launched and its report and the failed pushes are on the envelope; a report with a stop still launches, with the stop in the start prompt's summary line; an unreachable report launches with the warning; a reconcile error launches with the error reported; an already-live strand still runs the reconcile and reports it.
`newStartHarness` fills the new seam with a recording fake returning an empty report, so every existing `TestStart_*` keeps passing and the new test swaps the fake per row.

**ImpactSummary:** One new seam on `orchCLI` and one step in `startCmd`; every existing start path keeps its envelope keys and gains two.

**Commit:** 19: orch start runs the hub's origin reconcile first
